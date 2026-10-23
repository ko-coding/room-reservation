# room-reservation（会議室予約システム）

会議室の登録・予約を行う Web アプリケーション（研修で開発中）。

## 技術スタック

- Java 17 / Spring Boot
- PostgreSQL 17 / Flyway
- Docker Compose

## 起動方法

1. `.env.example` をコピーして `.env` を作成し、`DB_PASSWORD` を設定する
2. DB を起動する：`docker compose up -d`
3. アプリを起動する：`./mvnw spring-boot:run`
4. 動作確認：http://localhost:8080/actuator/health
