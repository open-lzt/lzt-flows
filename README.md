# lzt-flows

Каталог готовых модулей для движка автоматизаций [auto-lzt](https://github.com/open-lzt/auto-lzt).

Из бота:

```
/modules              — список доступных модулей
/import bump-daily    — поставить модуль по имени
```

С сервера, где стенд ещё не настроен:

```bash
wget -qO- https://github.com/open-lzt/open-lzt/raw/main/install-flow.sh | sudo bash
```

## Модули

Девять штук: семь графов и два пакета узлов.

| Модуль | Версия | Тип | Что делает |
|---|---|---|---|
| `bump-daily` | 1.1.0 | flow | поднимает все лоты аккаунта по расписанию, по умолчанию каждые 4 часа |
| `sniper-autobuy` | 1.0.0 | flow | автопокупка в любой категории — раздел параметром, а не зашит в граф |
| `steam-autobuy` | 2.1.0 | flow | автопокупка Steam: потолок цены, отсечка по количеству |
| `fortnite-autobuy` | 1.0.0 | flow | то же для Fortnite |
| `riot-autobuy` | 1.0.0 | flow | то же для Valorant и League of Legends |
| `supercell-autobuy` | 1.0.0 | flow | то же для Brawl Stars, Clash of Clans, Clash Royale |
| `telegram-autobuy` | 1.0.0 | flow | то же для Telegram-аккаунтов |
| `pricing-pack` | 1.0.0 | python | узлы ценообразования: процент от цены, округление до красивого числа |
| `notify-pack` | 1.0.0 | python | узлы уведомлений: Discord и произвольный вебхук |

У всех autobuy-модулей `dry_run` включён по умолчанию. Выключайте, когда посмотрели, что попадает в выдачу.

## Два типа модулей

`kind: python` — **это исполняемый код, который ставится на хост оператора через pip и работает с его токенами.** Поэтому такие модули публикует только владелец репозитория. `kind: flow` — данные, публикует кто угодно.

| | `kind: flow` | `kind: python` |
|---|---|---|
| Что это | граф узлов | пакет Python-узлов |
| Файлы | `module.yaml` + `flow.json` | `module.yaml` + `pyproject.toml` + пакет |
| Исполняется | движком, по графу | на хосте, как обычный код |
| Кто публикует | любой автор из `authors.yml` | только владелец репозитория |

## Формат модуля

`modules/<name>/module.yaml`:

```yaml
schema_version: 1
name: steam-autobuy
version: 2.1.0
author: zlexdev
kind: flow
description: |-
  Автопокупка Steam-аккаунтов: поиск с потолком цены, отсечка по количеству, покупка каждого.
  dry_run включён по умолчанию.
requires_nodes:
  - market.search
  - logic.condition
  - logic.take
  - logic.for_each_lot
  - market.fast_buy
```

`kind` можно опустить — по умолчанию `flow`. `author` обязан совпадать с автором PR.

`modules/<name>/flow.json` — сам граф: параметры и узлы.

```json
{
  "name": "steam-autobuy",
  "entry_node_id": "search",
  "params": [
    {
      "key": "max_price",
      "label": "Цена до",
      "control": "number",
      "required": true,
      "default": 10,
      "minimum": 1
    }
  ],
  "nodes": [
    {
      "id": "search",
      "type": "market.search",
      "inputs": {
        "max_price": { "literal": "{{vars.max_price}}" },
        "category": { "literal": "steam" }
      },
      "edges": { "next": "found" }
    }
  ]
}
```

## index.json

Генерируется автоматически при пуше в `main`. Руками не трогать — PR, меняющий его, отклоняется.

```json
{
  "schema_version": 1,
  "modules": [
    { "name": "steam-autobuy", "version": "2.1.0", "sha256": "98cf4a6a…", "kind": "flow" }
  ]
}
```

Хешируется `flow.json` для `kind: flow` и `pyproject.toml` для `kind: python`.

**sha256 здесь — проверка целостности передачи, а не подпись.** Она ловит битую загрузку, а не подмену: кто контролирует `index.json`, контролирует и хеш.

## Добавить свой модуль

1. Отдельный PR: добавьте свой GitHub-логин в `authors.yml`.
2. Отдельный PR: каталог `modules/<name>/`. Один модуль на PR, смешивать с `authors.yml` нельзя.
3. CI прогонит проверки.

Что гоняет `.github/workflows/validate.yml`:

| Проверка | Что валит PR |
|---|---|
| нет `pull_request_target` в workflows | защита от supply-chain: форк не должен получить секреты |
| `check_code_owner.py` | `.py` или `pyproject.toml` от кого-либо, кроме владельца репозитория |
| одна забота на PR | одновременно изменены `authors.yml` и `modules/` |
| `index.json` не тронут | ручная правка сгенерированного файла |
| `check_author.py` | `author` в манифесте не совпадает с автором PR или его нет в `authors.yml` |
| `lzt-flow-validate modules/<name>` | граф не компилируется или ссылается на неизвестный узел |

Валидатор — тот же самый, что стоит на бэкенде: ставится `pip install lzt-flow` из [auto-lzt](https://github.com/open-lzt/auto-lzt).

Настоящая граница безопасности для кода — не CI, а `CODEOWNERS` плюс защита ветки. Скрипт `check_code_owner.py` только даёт понятную ошибку раньше.

## Экосистема

[auto-lzt](https://github.com/open-lzt/auto-lzt) — движок · [lzt-plugins](https://github.com/open-lzt/lzt-plugins) — исполняемые плагины · [весь стенд](https://github.com/open-lzt/open-lzt)
