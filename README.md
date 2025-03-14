#Outline_Docker_Deployment

Проект по развёртыванию системы Outline в Docker-контейнерах с использованием изолированных сетей и доступа через Tor-сервис.

Цели проекта:

Развернуть Outline как изолированное приложение в Docker.
Организовать доступ к Outline исключительно через Tor (.onion домен).
Обеспечить безопасность с помощью изолированных Docker-сетей.
Подготовить структуру данных для удобного резервного копирования.
Архитектура и компоненты:

outline-app: веб-приложение Outline.
outline-postgres: база данных PostgreSQL.
outline-redis: кэш Redis.
outline-tor: сервис Tor, публикующий onion-домен.
Безопасность и сети: Создаются две изолированные сети:

outline-universe — публичная сеть с доступом в интернет (для Tor-сервиса).
outline-private — приватная сеть для связи между сервисами (без выхода в интернет).
Сервисы outline-app, outline-postgres и outline-redis работают исключительно внутри outline-private.
Сервис outline-tor работает в outline-universe и предоставляет доступ к Outline через onion-домен в Tor-сети.

Хранение данных и резервное копирование: Все данные и конфигурации сервисов хранятся в директории .vol в корне проекта для удобного резервного копирования и восстановления данных.

Структура каталогов:

/outline-docker
docker-compose.yml
.env
.vol/
outline-tor/var/tor/ (данные Tor-сервиса)
outline-postgres/ (данные PostgreSQL)
outline-redis/ (данные Redis, если требуется)
README.md
Примеры монтирования томов:

.vol/outline-tor/var/tor монтируется в /var/tor контейнера outline-tor.
.vol/outline-postgres монтируется в /var/lib/postgresql/data контейнера outline-postgres.
.vol/outline-redis монтируется в /data контейнера outline-redis (если используется).
Tor и Onion-домен:

Сервис outline-tor публикует Onion-домен для доступа к приложению Outline.
Протокол внутри Tor-сети — HTTP. HTTPS не требуется, так как шифрование обеспечивается Tor.
Пробрасывается порт 3000 с outline-app на порт 80 сервиса outline-tor (доступ по onion-домену).
Docker Compose: сети и сервисы: outline-app работает в сети outline-private.
outline-postgres работает в сети outline-private.
outline-redis работает в сети outline-private.
outline-tor работает в сети outline-universe.

Требования к именованию:

Все сервисы должны иметь префикс outline-.
Все сети должны иметь префикс outline-.
Пример Docker Compose конфигурации:

version: '3.8'

services: outline-app: image: outlinewiki/outline:latest depends_on: - outline-postgres - outline-redis env_file: .env networks: - outline-private restart: unless-stopped volumes: - ./vol/outline-app/uploads:/var/lib/outline/uploads

outline-postgres: image: postgres:14 environment: POSTGRES_USER: outline POSTGRES_PASSWORD: password POSTGRES_DB: outline volumes: - ./vol/outline-postgres:/var/lib/postgresql/data networks: - outline-private restart: unless-stopped

outline-redis: image: redis:6 volumes: - ./vol/outline-redis:/data networks: - outline-private restart: unless-stopped

outline-tor: image: goldy/tor-hidden-service environment: VIRTUAL_PORT: 3000 HIDDEN_SERVICE_VERSION: 3 HIDDEN_SERVICE_PORTS: 80:3000 volumes: - ./vol/outline-tor/var/tor:/var/lib/tor networks: - outline-universe restart: unless-stopped

networks: outline-universe: driver: bridge outline-private: driver: bridge internal: true

План разработки и задачи (Issues):

Создать репозиторий и базовую структуру проекта.
Добавить изолированные сети outline-universe и outline-private.
Добавить Tor-сервис с onion-доменом.
Настроить тома и хранилища данных.
Привести все имена сервисов и сетей к формату outline-*.
Добавить документацию README.md.
(Опционально) Настроить CI/CD через GitHub Actions для автодеплоя.
Резервное копирование: Для резервного копирования необходимо заархивировать папку .vol:

tar -czvf outline-vol-backup.tar.gz .vol/

Полезные ссылки:

Outline Wiki GitHub: https://github.com/outline/outline
Tor Project: https://www.torproject.org/
Docker Networking: https://docs.docker.com/network/
Дополнительно: В продакшн-окружении рекомендуется ограничить доступ к сервисам, использовать обратный прокси, а также настроить регулярное резервное копирование.
