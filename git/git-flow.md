### Создание функциональной ветки

```bash
git checkout develop
git checkout -b feature_branch
```

### Окончание работы с функциональной веткой

```bash
git checkout develop
git merge feature/feature_branch
# Удаление feature-ветки локально
git branch -d feature/feature_branch
# (Опционально) Удаление удалённой feature-ветки
git push origin --delete feature/feature_branch
```