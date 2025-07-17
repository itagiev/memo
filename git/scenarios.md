### Случайные внесения изменений не в ту ветку

К примеру мы в f-b, изменили или создали файл d.txt

```bash
# Фиксируем изменения в буфер
git stash -u
```

```bash
# Если ветки нет, тогда
git checkout main
git checkout -b f-d
```

Если ветка уже есть `git checkout f-d`

И возвращаем изменения из stash `git stash pop`

А если мы уже успели сделать коммит, тогда можно отменить их `git reset HEAD~1`

### Merge conflicts

Допустим мы хотим слить ветку feature-a в main, но получаем conflict

#### Merge

```bash
# Переходим в ветку feature-a
git checkout feature-a
# Скачиваем обновления
git fetch
# Делаем merge
git merge origin/main
# Разрешаем конфликт, делаем коммит и пушим
git add -A
git commit -m "Merge branch 'main' into branch 'feature-a'"
git push
```

#### Rebase

```bash
# Переходим в ветку feature-a
git co feature-a
# Скачиваем обновления
git fetch
# Делаем rebase
git rebase origin/main
# Разрешаем конфликт и делаем коммит
git add -A
git commit "Update feature-a"
git rebase --continue
# Если был rebase, тогда мы как бы меняем истоию и нужно делать force push
git push -f
```

### История переписана (git push -f)

При совершении коммита с помощью `--amend` и `git push -f`, история будет переписана

В локальных проектах желательно сделать

```bash
git fetch origin
git reset --hard origin/main # Это удалит все незакоммиченные изменения
```