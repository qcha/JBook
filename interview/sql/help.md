# Помощь по разделу 'sql задачи для интервью'

## Где проверять/запускать задачи и решения

### Db-fiddle

В качестве инструмента для решения задач по данной теме я рекомендую [db-fiddle](https://www.db-fiddle.com).

### Docker-Compose

При необходимости (например, в задачах в которых есть `init-db.sql` скрипты или предоставляются демо базы) можно воспользоваться простым `docker-compose` объявлением сервисов:

```yaml
services:
  postgres:
    container_name: example-db
    image: postgres:15.8
    environment:
      POSTGRES_USER: "user"
      POSTGRES_PASSWORD: "qwerty1234"
    volumes:
      - .:/docker-entrypoint-initdb.d
    ports:
      - "5432:5432"
    networks:
      - postgres

  pgadmin:
    container_name: pgadmin_container
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "user@user.com"
      PGADMIN_DEFAULT_PASSWORD: "user"
      PGADMIN_CONFIG_SERVER_MODE: "False"
    volumes:
      - .:/var/lib/pgadmin
    ports:
      - "5050:80"
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 1G
    networks:
      - postgres

networks:
  postgres:
    driver: bridge
```

Предоставит БД и админку для запросов.

Если скрипт инициализации БД будет в той же директории, что и `docker-compose` файл, он будет применен автоматически на старте контейнера.

## Общие советы по написанию запросов

Общие принципы и советы для оптимизации произовдителности решений:

* Сначала фильтруйте данные, потом `join`
* Сначала фильтруйте данные, потом группируйте
* Оператор `in` используйте **только** для маленьких множеств
* Старайтесь в `join` использовать `equi join`: сравнение через равенство
* Подзапросы зачастую не лучшее решение, как вариант: выносить в CTE (но тоже аккуратно)
* Не использовать `union`
* Смотрите план запроса через `explain` и `explain analyze`
* Один и тот же запрос на разных объемах данных и в разных средах исполнения может давать разную производительность, поэтому преждевременно не оптимизируйте запросы
