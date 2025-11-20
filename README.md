## Главный функционал

Администратор платформы (заказчик) загружает Excel-таблицу с необходимыми позициями и количеством каждой из них, после чего конфигурирует и публикует заказ. Опубликованный заказ становится доступен пользователям — поставщикам.

Поставщики могут предложить свои цены на поставку позиций. После сбора всех цен (или в момент, который определит администратор) запускается механизм торгов.

Во время торгов у поставщиков рядом с каждой предложенной ценой отображается индикатор, показывающий, является ли их цена лучшей среди конкурентов. На этом этапе поставщик может снизить свою цену, чтобы улучшить позицию.

После завершения торгов заказ закрывается для редактирования. Администратор анализирует результаты и автоматически формирует отдельные заказы для каждого поставщика, выбирая самые выгодные предложения.

Кроме того, администратор может гибко настроить процесс формирования персональных заказов: например, игнорировать предложения отдельных поставщиков или, наоборот, отдать предпочтение поставщику независимо от его цены.

## Технологии


- **ORY** — авторизация и управление доступом.  
- **PostgreSQL** — отдельная база данных для каждого сервиса.  
- **ClickHouse** — используется для хранения и анализа истории изменения цен.  
- **Redis** — кеширование и ускорение операций.  

Каждый сервис имеет классические REST-эндпоинты и, при необходимости, может взаимодействовать с другими сервисами через **gRPC** или **RabbitMQ** в зависимости от задачи.

- **S3-хранилище** — загрузка и хранение Excel-файлов.

Деплой предполагается в **Kubernetes** или **Docker Swarm**, также поддерживается запуск через обычный **Docker Compose**.

- **Prometheus** и **Grafana** — сбор и визуализация метрик.

## Примерная архитектура
```mermaid
%% Финальная архитектура с зонами доступа
graph LR
    %% ========= ЗОНА 1: ФРОНТ И ПУБЛИЧНЫЙ ДОСТУП =========
    subgraph PUBLIC_ZONE["Public Zone (доступно фронтенду)"]
        A1["Load balancer"]
    end

    %% ========= ЗОНА 2: ПРИЛОЖЕНИЯ (бизнес-логика) =========
    subgraph APP_ZONE["Application Zone (микросервисы)"]
        A["Frontend (React)"]
        B["API Gateway"]

        C["Auth Service (ORY)"]
        U["Users Service"]
        D["Order Service"]
        E["Pricing Service"]
        F["Participant Service"]
        G["Bidding Service"]
        H["Excel Service"]
        I["Files Service"]
        J["PersonalOrder Service"]
        K["Notification Service"]
        Z["Analytics Service"]
        X["Items Service"]
        Y["OrderItems Service"]

        %% Связи от Gateway
        B -->|"Auth / JWT"| C
        B -->|"Users"| U
        B -->|"Orders"| D
        B -->|"Order items"| Y
        B -->|"Items"| X
        B -->|"Prices"| E
        B -->|"Participants"| F
        B -->|"Bidding"| G
        B -->|"Excel Upload / Download"| H
        B -->|"Files (Presigned URLs)"| I
        B -->|"Pers. orders"| J
        B -->|"Notifications / Webhooks"| K
        B -->|"Analytics"| Z
    end

    %% Генеральные входы
    A1 -->|"HTTPS"| A
    A1 -->|"REST/gRPC"| B

    %% ========= ЗОНА 3: ИНФРАСТРУКТУРА =========
    subgraph INFRA_ZONE["Infrastructure Zone"]
        M["PostgreSQL (per service)"]
        N["Redis (Cache / Sessions)"]
        L["MinIO (S3 Storage)"]
        Q["RabbitMQ"]
        T["Prometheus + Grafana"]
    end

    %% Связи с PostgreSQL
    U -->|"PostgreSQL"| M
    D -->|"PostgreSQL"| M
    E -->|"PostgreSQL"| M
    F -->|"PostgreSQL"| M
    G -->|"PostgreSQL"| M
    I -->|"PostgreSQL"| M
    J -->|"PostgreSQL"| M
    K -->|"PostgreSQL"| M
    X -->|"PostgreSQL"| M
    Y -->|"PostgreSQL"| M 

    %% S3
    I -->|"S3"| L
```

## Примерная структура репы

```
goodprice-tender-platform/
│
├── api-gateway/
|
├── frontend/
|
├── auth-service/
├── users-service/
├── order-service/
├── orderitems-service/
├── items-service/
├── pricing-service/
├── participant-service/
├── bidding-service/
├── excel-service/
├── files-service/
├── personal-orders-service/
├── notification-service/
├── analytics-service/
├── shared-go/                   #Возможно не будет
├── shared-python/
|
├── infra/                     
├── docs/
└── devtools/
```