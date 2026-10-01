# Домашнее задание к занятию «Ветвления в Git»

## Ссылки для проверки

- Репозиторий: https://github.com/kinnwailos/hm-branching-git
- **Network-график:** https://github.com/kinnwailos/hm-branching-git/network
- Скрипты: [branching/merge.sh](branching/merge.sh), [branching/rebase.sh](branching/rebase.sh)

## Цель

Потренироваться в `merge` и `rebase`, увидеть разницу между ними и разрешить конфликты, когда две ветки меняют один и тот же участок кода.

## Шаг 1. Подготовка файлов

В каталоге `branching` созданы `merge.sh` и `rebase.sh`. Изначально оба скрипта печатают все аргументы одной строкой (`"$*"`).

Коммит в `main`: **prepare for merge and rebase**.

## Ветка git-merge

От `main` создана ветка `git-merge`.

1. Параметры выводятся по одному через `"$@"` — коммит **merge: @ instead ***.
2. Цикл заменён на `while` + `shift` — коммит **merge: use shift**.

Пока шла работа в `git-merge`, в `main` изменили `rebase.sh` (тоже `"$@"`, плюс разделитель `=====`) и запушили ветку.

## Ветка git-rebase

Ветка `git-rebase` создана не от актуального `main`, а от коммита **prepare for merge and rebase** — как если бы разработчик не сделал `git pull`.

1. Коммит **git-rebase 1**: вывод `Parameter: $param`.
2. Коммит **git-rebase 2**: вывод `Next parameter: $param`.

На этом этапе в истории одновременно живут `main`, `git-merge` и «отставшая» `git-rebase`.

## Merge

```text
git checkout main
git merge git-merge
```

Конфликтов не было: ветки трогали разные файлы (`merge.sh` vs `rebase.sh`). Git сделал merge-коммит стратегией ort/recursive.

После этого `main` содержит и правки `merge.sh` из `git-merge`, и правки `rebase.sh` из `main`.

## Rebase

Перед слиянием `git-rebase` в `main` выполнен интерактивный rebase:

```text
git checkout git-rebase
git rebase -i main
```

Два коммита ветки объединены: у нижнего указан `fixup`.

Возникли ожидаемые конфликты в `branching/rebase.sh`:

1. Первый коммит: оставлен вариант `main` — `echo "$@ Parameter #$count = $param"`.
2. Второй коммит: оставлен вариант ветки — `echo "Next parameter: $param"`.

После `git add` и `git rebase --continue` история `git-rebase` переписана: оба коммита стали одним поверх актуального `main`.

Обычный `git push` отклонён (non-fast-forward): локальная ветка больше не продолжает старый remote. Как в задании, история обновлена принудительно:

```text
git push -u origin git-rebase -f
```

## Fast-forward в main

```text
git checkout main
git merge git-rebase
```

Слияние прошло **fast-forward**, без дополнительного merge-коммита: `git-rebase` уже стояла поверх `main`.

## Итоговая схема

```text
* git-rebase 1                    (main, git-rebase)
*   Merge branch 'git-merge'
|\
| * merge: use shift              (git-merge)
| * merge: @ instead *
* | main: rebase.sh @ instead *
|/
* prepare for merge and rebase
```

- **merge** сохраняет параллельные коммиты и добавляет узел слияния.
- **rebase** переносит свои коммиты на новый базовый коммит, переписывает хеши и при `fixup` склеивает промежуточные шаги в один.

## Скриншоты

| Что | Файл |
|-----|------|
| Network-график GitHub | [screenshots/network.png](screenshots/network.png) |
| Итоговый `git log --graph` | [screenshots/git-log-graph.png](screenshots/git-log-graph.png) |
