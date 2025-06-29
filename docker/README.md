## Работа с Docker

CLI команды, обучающие материалы, примеры

### CLI

#### Пример запуска asp.net приложения

```
docker run --rm [-d] -p 80:8080 --name app app:latest
```

#### Открыть bash терминал внутри контейнера

```
docker exec -it <имя контейнера> /bin/bash
```

#### Сокращение для интерактивного режима и запуска на заднем фоне

```
docker run -itd
```

#### Clear build cache

```
docker builder prune -af
```

#### Удалить контейнер, включая сеть, (volume продолжить существовать)

```
docker compose down
```

### Ссылки

- [Docker documentation / Guides](https://docs.docker.com/guides/)
- [Docker networking is CRAZY!! (you NEED to learn it)](https://www.youtube.com/watch?v=bKFMS5C4CG0&ab_channel=NetworkChuck)
