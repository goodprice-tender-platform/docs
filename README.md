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