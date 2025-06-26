### git add

Добавить все в какой бы директории репозитория мы не находились

```bash
git add -A
```

### git commit

Сделать комит staged элементов в предыдущий комит, в истории не появится новой записи

```bash
git commit --amend --no-edit
```

### git reset

Отменить последний комит и изменить история так, как будто последнего комита никогда и не было \
Самый первый комит, отменить невозможно

```bash
git reset HEAD~1
# Отменить послендие 2, 3, n комитов
git reset HEAD~2
git reset HEAD~3
```

Отменить изменения во все отслеживаемых и не отслеживаемых файлах, новые файлы буду удалены \
то есть мы полностью откатываемся к последнему комиту

```bash
git reset --hard
```

### git revert

Отменить комит, но при этом он останется в истории

```bash
git revert 6ef5eb5
```

### git stash

При создании ветки, если мы не хотим делать комит текущих изменений \
Создается как бы виртуальная ветка, которая содержит изменения и git status выдает чистый результат

```bash
git stash
# Чтобы захватить untracked файлы
git stash -u
# Просмотреть stash
git stash list
# stash@{0}: WIP on main: c3a3126 adding b.txt

# Вернуть изменения из stash
git stash pop
```

### git clean

Удалить все неотслеживаемые файлы

```bash
git clean -f
# Чтобы удалить с директориями
git clean -fd
```

### git checkout

Создать новую ветку от текущей

```bash
git checkout -b feature-MyNewFeature
```

Прыгнуть на существующую ветку

```bash
git checkout main
```

### git merge

Слияние веток

```bash
# Переходим в ветку в которую нужно слить другую ветку
git checkout main
git merge feature-MyNewFeature

# Слияние с удаленной веткой после git fetch
git merge origin/main
```

### git branch

Список веток

```bash
git branch --list
# или
git branch
# или удаленные ветки
git branch --list -r
# или все
git branch --list -a
```

Удаление веток

```bash
git branch -D feature-MyNewFeature
```

Сделать ветку главной
```bash
git branch -M main
```

### git remote

Добавить адрес удаленного репозитория

```bash
git remote add origin git@github.com:username/project-name.git

# Посмотреть список remote
git remote -v
```

### git push

Отправить комит в удаленный репозиторий

```bash
# -u - установит текущий удаленный репозиторий в состояние upstream
git push -u origin main
```

### git restore

Unstage зафиксированного элемента

```bash
git restore --staged filename.txt
```

Отмена внесенных изменений в файл

```bash
git restore filename.txt
```

### git fetch

Скачать обновления ветки

```bash
git fetch
```

### git pull

Скачать все обновления ветки и сделать merge

```bash
# Скачать все обновления с remote который установлен по умолчанию
git pull
# равносильно git pull origin main
```
