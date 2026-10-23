# room-reservation（会議室予約システム）

会議室の登録・予約を行う Web アプリケーション（研修で開発中）。

## 技術スタック

- Java 17 / Spring Boot
- PostgreSQL 17 / Flyway
- Docker Compose

## 前提条件

- Java 17
- Docker Desktop（Docker Compose が使えること）

## 起動方法

1. `.env.example` をコピーして `.env` を作成し、`DB_PASSWORD` を設定する
   - macOS / Linux：`cp .env.example .env`
   - Windows（PowerShell）：`Copy-Item .env.example .env`

2. DB を起動する：`docker compose up -d`
3. アプリを起動する：
   - macOS / Linux：`./mvnw spring-boot:run`
   - Windows（PowerShell）：`.\mvnw.cmd spring-boot:run`

4. 動作確認：http://localhost:8080/actuator/health
