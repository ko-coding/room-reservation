## API一覧

| #   | メソッド | パス                   | 説明                   | 認証 | 権限        | 成功 | 主なエラー              |
| --- | -------- | ---------------------- | ---------------------- | ---- | ----------- | ---- | ----------------------- |
| 1   | POST     | /api/auth/register     | ユーザー登録           | 不要 | -           | 201  | 400, 409                |
| 2   | POST     | /api/auth/login        | ログイン               | 不要 | -           | 200  | 400, 401                |
| 3   | GET      | /api/users             | ユーザー一覧取得       | 必要 | ADMIN       | 200  | 400, 401, 403           |
| 4   | GET      | /api/users/{id}        | ユーザー詳細情報取得   | 必要 | ADMIN, USER | 200  | 401, 403, 404           |
| 5   | PUT      | /api/users/{id}        | ユーザー情報更新       | 必要 | USER        | 200  | 400, 401, 404           |
| 6   | DELETE   | /api/users/{id}        | ユーザー情報削除       | 必要 | ADMIN       | 204  | 401, 403, 404, 409      |
| 7   | POST     | /api/rooms             | 会議室登録             | 必要 | ADMIN       | 201  | 400, 401, 403           |
| 8   | GET      | /api/rooms             | 会議室一覧取得         | 必要 | ADMIN, USER | 200  | 401                     |
| 9   | GET      | /api/rooms/{id}        | 会議室詳細情報取得     | 必要 | ADMIN, USER | 200  | 401, 404                |
| 10  | PUT      | /api/rooms/{id}        | 会議室情報更新         | 必要 | ADMIN       | 200  | 400, 401, 403, 404      |
| 11  | DELETE   | /api/rooms/{id}        | 会議室情報削除         | 必要 | ADMIN       | 204  | 401, 403, 404, 409      |
| 12  | POST     | /api/reservations      | 会議室予約             | 必要 | ADMIN, USER | 201  | 400, 401, 403, 409      |
| 13  | GET      | /api/reservations      | 会議室予約一覧取得     | 必要 | ADMIN, USER | 200  | 401                     |
| 14  | GET      | /api/reservations/{id} | 会議室予約詳細情報取得 | 必要 | ADMIN, USER | 200  | 401, 404                |
| 15  | PATCH    | /api/reservations/{id} | 予約キャンセル         | 必要 | ADMIN, USER | 200  | 400, 401, 403, 404, 409 |

## 共通事項

- 認証は `Authorization: Bearer <token>` で送る
- 日時は `ISO 8601`
- エラー本文に、スタックトレースやSQLなどの内部情報は含めない

### エラーレスポンス形式

```json
{
  "code": "RESERVATION_CONFLICT",
  "message": "予約の時間帯が重なっています"
}
```

```json
{
  "code": "VALIDATION_ERROR",
  "message": "入力エラー",
  "details": [
    {
      "field": "title",
      "message": "必須です"
    },
    {
      "field": "endAt",
      "message": "終了は開始より後にしてください"
    }
  ]
}
```

### code 一覧

| code                           | HTTP | 発生する場面                                     |
| ------------------------------ | ---- | ------------------------------------------------ |
| VALIDATION_ERROR               | 400  | 入力値の型不正、必須漏れなど                     |
| BUSINESS_RULE_VALIDATION       | 400  | 営業時間外、30分刻みでない、期間外、過去など     |
| UNAUTHENTICATED                | 401  | トークンがない・無効                             |
| INVALID_CREDENTIALS            | 401  | ログインでメールかパスワードが違う               |
| ACCESS_DENIED                  | 403  | 他人の情報を操作しようとした                     |
| USER_NOT_FOUND                 | 404  | 指定したidのuserが見つからない                   |
| ROOM_NOT_FOUND                 | 404  | 指定したidのroomが見つからない                   |
| RESERVATION_NOT_FOUND          | 404  | 指定したidのreservationが見つからない            |
| EMAIL_CONFLICT                 | 409  | emailの重複                                      |
| RESERVATION_CONFLICT           | 409  | 予約の時間帯が重なっている                       |
| USER_HAS_RESERVATION           | 409  | 予約が残っているユーザーを削除しようとした       |
| ROOM_HAS_RESERVATION           | 409  | 予約が残っている会議室を削除しようとした         |
| RESERVATION_CANCEL_NOT_ALLOWED | 409  | キャンセルの期限（開始の24時間前）を過ぎている   |
| RESERVATION_ALREADY_CANCELED   | 409  | 既にキャンセル済みの予約をキャンセルしようとした |
| INTERNAL_SERVER_ERROR          | 500  | サーバー内部の想定外エラー                       |

## API詳細

### 1. POST /api/auth/register（ユーザー登録）

- 認証：不要
- 説明：未ログインの人が自分のアカウントを作成するAPI。このAPIで登録したユーザーの`role`は`USER`となる（管理者への昇格を防ぐためクライアントからは指定できない。）。メールアドレスは保存前に小文字に統一する。

**リクエスト**

```json
{
  "email": "user02@example.com",
  "password": "password",
  "name": "user02"
}
```

**入力の制約**

| 項目     | 必須 | 制約                                               |
| -------- | ---- | -------------------------------------------------- |
| email    | ○    | メールアドレス形式、1～255文字、空文字を許容しない |
| password | ○    | 8～72バイト・空文字を許容しない                    |
| name     | ○    | 1～100文字・空文字を許容しない                     |

**レスポンス 201 Created**

```json
{
  "id": 2,
  "email": "user02@example.com",
  "name": "user02",
  "role": "USER",
  "createdAt": "2026-01-01T10:00:00+09:00",
  "updatedAt": "2026-01-01T10:00:00+09:00"
}
```

**エラー**

| ステータス | 条件                                                           | code             |
| ---------- | -------------------------------------------------------------- | ---------------- |
| 400        | 入力制約に違反している（必須項目の欠落、形式不正、文字数超過） | VALIDATION_ERROR |
| 409        | emailが重複している                                            | EMAIL_CONFLICT   |

### 2. POST /api/auth/login（ログイン）

- 認証：不要
- 説明：入力されたメールアドレスを小文字にして照合する。成功したらJWTトークンを返すAPI。取得したJWTトークンは認証が必要なAPIを使用する場合に `Authorizationヘッダー` に付与して送信する。

**リクエスト**

```json
{
  "email": "user02@example.com",
  "password": "password"
}
```

**入力の制約**

| 項目     | 必須 | 制約                                   |
| -------- | ---- | -------------------------------------- |
| email    | ○    | メールアドレス形式、空文字を許容しない |
| password | ○    | 空文字を許容しない                     |

**レスポンス 200 OK**

```json
{
  "token": "xxx.yyy.zzz"
}
```

**エラー**

| ステータス | 条件                                                                       | code                |
| ---------- | -------------------------------------------------------------------------- | ------------------- |
| 400        | 入力制約に違反している（必須項目の欠落、形式不正）                         | VALIDATION_ERROR    |
| 401        | メールアドレスかパスワードが違う（登録済みメールアドレスの推測を防ぐため） | INVALID_CREDENTIALS |

### 3. GET /api/users（ユーザー一覧取得）

- 認証：必要（ADMIN）
- 説明：ADMINのみ操作可能。ユーザーを条件で絞り込み、ページ単位で返す。並び順は `id` の昇順で固定とする。

**クエリパラメータ**

| 名前  | 型      | 必須 | 既定値 | 説明                                                 |
| ----- | ------- | ---- | ------ | ---------------------------------------------------- |
| page  | integer | 任意 | 0      | ページ番号（0 始まり）。0 以上                       |
| size  | integer | 任意 | 20     | 1 ページの件数。1〜100                               |
| email | string  | 任意 | なし   | メールアドレスの部分一致（大文字小文字は区別しない） |
| name  | string  | 任意 | なし   | 名前の部分一致（大文字小文字は区別しない）           |
| role  | string  | 任意 | なし   | ロールの完全一致（大文字小文字は区別しない）         |

**リクエスト例**

```
GET /api/users?page=0&size=20&email=example.com&name=user&role=USER
```

**レスポンス 200 OK**

```json
{
  "content": [
    {
      "id": 2,
      "email": "user02@example.com",
      "name": "user02",
      "role": "USER",
      "createdAt": "2026-01-01T10:00:00+09:00",
      "updatedAt": "2026-01-01T10:00:00+09:00"
    },
    {
      "id": 3,
      "email": "user03@example.com",
      "name": "user03",
      "role": "USER",
      "createdAt": "2026-01-01T10:00:00+09:00",
      "updatedAt": "2026-01-01T10:00:00+09:00"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 2,
  "totalPages": 1
}
```

**エラー**

| ステータス | 条件                                                                                  | code             |
| ---------- | ------------------------------------------------------------------------------------- | ---------------- |
| 400        | `page` が負、`size` が 1〜100 の範囲外、`role` が `USER` / `ADMIN` 以外、数値でない値 | VALIDATION_ERROR |
| 401        | トークンなし・無効                                                                    | UNAUTHENTICATED  |
| 403        | ADMINでないロールのユーザーが操作しようとした                                         | ACCESS_DENIED    |

### 4. GET /api/users/{id}（ユーザー詳細情報取得）

- 認証：必要（ADMIN, USER）
- 説明：パスパラメータで指定したIDのユーザー詳細情報を取得する。`USER` は自分の情報のみ取得可能。他人のユーザーIDを指定した場合は 403 を返す。IDは連番で存在を隠す必要性が低いため、404 には統一しない。（機密性が高いリソースでは 404 に統一する選択肢もある）

**リクエスト例**

```
GET /api/users/1
```

**レスポンス 200 OK**

```json
{
  "id": 1,
  "email": "user01@example.com",
  "name": "user01",
  "role": "ADMIN",
  "createdAt": "2026-01-01T10:00:00+09:00",
  "updatedAt": "2026-01-01T10:00:00+09:00"
}
```

**エラー**

| ステータス | 条件                               | code            |
| ---------- | ---------------------------------- | --------------- |
| 401        | トークンなし・無効                 | UNAUTHENTICATED |
| 403        | USERが他人の情報を取得しようとした | ACCESS_DENIED   |
| 404        | 指定したidのuserが見つからない     | USER_NOT_FOUND  |

### 5. PUT /api/users/{id}（ユーザー情報更新）

- 認証：必要（USER）
- 説明：

  **リクエスト**

```json
{
  "name": "user999"
}
```

**レスポンス 200 OK**

```json
{
  "id": 2,
  "email": "user02@example.com",
  "name": "user999",
  "role": "USER",
  "createdAt": "2026-01-01T10:00:00+09:00",
  "updatedAt": "2026-01-02T10:00:00+09:00"
}
```

**エラー**

| ステータス | 条件                     | code             |
| ---------- | ------------------------ | ---------------- |
| 400        | 入力値不正               | VALIDATION_ERROR |
| 401        | トークンなし・無効       | UNAUTHENTICATED  |
| 404        | 対象リソースが存在しない |                  |

### 6. DELETE /api/users/{id}（ユーザー情報削除）

- 認証：必要（ADMIN）
- 説明：

**エラー**

| ステータス | 条件                                       | code                 |
| ---------- | ------------------------------------------ | -------------------- |
| 401        | トークンなし・無効                         | UNAUTHENTICATED      |
| 403        | 参照権限不足                               | ACCESS_DENIED        |
| 404        | 対象リソースが存在しない                   |                      |
| 409        | 予約が残っているユーザーを削除しようとした | USER_HAS_RESERVATION |

### 7. POST /api/rooms（会議室登録）

- 認証：必要（ADMIN）
- 説明：

**リクエスト**

```json
{
  "name": "room_01",
  "description": "test room",
  "capacity": 10
}
```

**レスポンス 201 Created**

```json
{
  "id": 1,
  "name": "room_01",
  "description": "test room",
  "capacity": 10,
  "createdAt": "2026-01-01T10:00:00+09:00",
  "updatedAt": "2026-01-01T10:00:00+09:00"
}
```

**エラー**

| ステータス | 条件               | code             |
| ---------- | ------------------ | ---------------- |
| 400        | 入力値不正         | VALIDATION_ERROR |
| 401        | トークンなし・無効 |                  |
| 403        | 参照権限不足       | ACCESS_DENIED    |

### 8. GET /api/rooms（会議室一覧取得）

- 認証：必要（ADMIN, USER）
- 説明：会議室を検索条件で絞り込み、ページ単位で返す。並び順は `id` の昇順で固定とする

**クエリパラメータ**

| 名前        | 型      | 必須 | 既定値 | 説明                                           |
| ----------- | ------- | ---- | ------ | ---------------------------------------------- |
| page        | integer | 任意 | 0      | ページ番号（0 始まり）。0 以上                 |
| size        | integer | 任意 | 20     | 1 ページの件数。1〜100                         |
| keyword     | string  | 任意 | なし   | 会議室名の部分一致（大文字小文字は区別しない） |
| minCapacity | integer | 任意 | なし   | この人数以上を収容できる会議室に絞る。1 以上   |

**リクエスト例**

```
GET /api/rooms?page=0&size=20&keyword=room&minCapacity=10
```

**レスポンス 200 OK**

```json
{
  "content": [
    {
      "id": 1,
      "name": "room_01",
      "description": "test room",
      "capacity": 10,
      "createdAt": "2026-01-01T10:00:00+09:00",
      "updatedAt": "2026-01-01T10:00:00+09:00"
    },
    {
      "id": 2,
      "name": "room_02",
      "description": "test room",
      "capacity": 20,
      "createdAt": "2026-01-01T10:00:00+09:00",
      "updatedAt": "2026-01-01T10:00:00+09:00"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 2,
  "totalPages": 1
}
```

**エラー**

| ステータス | 条件                                                                          | code             |
| ---------- | ----------------------------------------------------------------------------- | ---------------- |
| 400        | `page` が負、`size` が 1〜100 の範囲外、`minCapacity` が 1 未満、数値でない値 | VALIDATION_ERROR |
| 401        | トークンなし・無効                                                            | UNAUTHENTICATED  |

### 9. GET /api/rooms/{id}（会議室詳細情報取得）

- 認証：必要（ADMIN, USER）
- 説明：

**レスポンス 200 OK**

```json
{
  "id": 1,
  "name": "room_01",
  "description": "test room",
  "capacity": 10,
  "createdAt": "2026-01-01T10:00:00+09:00",
  "updatedAt": "2026-01-01T10:00:00+09:00"
}
```

**エラー**

| ステータス | 条件                     | code            |
| ---------- | ------------------------ | --------------- |
| 401        | トークンなし・無効       | UNAUTHENTICATED |
| 404        | 対象リソースが存在しない |                 |

### 10. PUT /api/rooms/{id}（会議室情報更新）

- 認証：必要（ADMIN）
- 説明：`name`, `capacity` は必須。`name` は 100 文字以内、`capacity` は 1 以上、`description` は 255 文字以内。`description` を省略すると `NULL` になる。

  **リクエスト**

```json
{
  "name": "room_update",
  "description": "test room_update",
  "capacity": 100
}
```

**レスポンス 200 OK**

```json
{
  "id": 1,
  "name": "room_update",
  "description": "test room_update",
  "capacity": 100,
  "createdAt": "2026-01-01T10:00:00+09:00",
  "updatedAt": "2026-01-02T10:00:00+09:00"
}
```

**エラー**

| ステータス | 条件               | code             |
| ---------- | ------------------ | ---------------- |
| 400        | 入力値不正         | VALIDATION_ERROR |
| 401        | トークンなし・無効 | UNAUTHENTICATED  |
| 403        | 参照権限不足       |                  |
| 404        | 会議室が存在しない |                  |

### 11. DELETE /api/rooms/{id}（会議室情報削除）

- 認証：必要（ADMIN）
- 説明：

**レスポンス 204 No Content**

**エラー**

| ステータス | 条件                                     | code |
| ---------- | ---------------------------------------- | ---- |
| 401        | トークンなし・無効                       |      |
| 403        | 参照権限不足                             |      |
| 404        | 対象リソースが存在しない                 |      |
| 409        | 予約が残っている会議室を削除しようとした |      |

### 12. POST /api/reservations（会議室予約）

- 認証：必要（ADMIN, USER）
- 説明：

**リクエスト**

```json
{
  "roomId": 1,
  "title": "meeting",
  "startAt": "2026-01-02T10:00:00+09:00",
  "endAt": "2026-01-02T10:30:00+09:00"
}
```

**レスポンス 201 Created**

```json
{
  "id": 1,
  "userId": 1,
  "roomId": 1,
  "title": "meeting",
  "startAt": "2026-01-02T10:00:00+09:00",
  "endAt": "2026-01-02T10:30:00+09:00",
  "status": "RESERVED",
  "createdAt": "2026-01-01T10:30:00+09:00",
  "updatedAt": "2026-01-01T10:30:00+09:00"
}
```

**エラー**

| ステータス | 条件                       | code                     |
| ---------- | -------------------------- | ------------------------ |
| 400        |                            | BUSINESS_RULE_VALIDATION |
| 401        | トークンなし・無効         |                          |
| 403        | 参照権限不足               |                          |
| 404        | 対象リソースが存在しない   |                          |
| 409        | 予約の時間帯が重なっている |                          |

### 13. GET /api/reservations（会議室予約一覧取得）

- 認証：必要（ADMIN, USER）
- 説明：

**レスポンス 200 OK**

```json
{
  "reservations": [
    {
      "id": 1,
      "userId": 1,
      "roomId": 1,
      "title": "meeting",
      "startAt": "2026-01-02T10:00:00+09:00",
      "endAt": "2026-01-02T10:30:00+09:00",
      "status": "RESERVED",
      "createdAt": "2026-01-01T10:30:00+09:00",
      "updatedAt": "2026-01-01T10:30:00+09:00"
    },
    {
      "id": 2,
      "userId": 2,
      "roomId": 2,
      "title": "meeting",
      "startAt": "2026-01-02T10:00:00+09:00",
      "endAt": "2026-01-02T10:30:00+09:00",
      "status": "RESERVED",
      "createdAt": "2026-01-01T10:30:00+09:00",
      "updatedAt": "2026-01-01T10:30:00+09:00"
    }
  ]
}
```

**エラー**

| ステータス | 条件               | code |
| ---------- | ------------------ | ---- |
| 401        | トークンなし・無効 |      |

### 14. GET /api/reservations/{id}（会議室予約詳細情報取得）

- 認証：必要（ADMIN, USER）
- 説明：

**レスポンス 200 OK**

```json
{
  "id": 1,
  "userId": 1,
  "roomId": 1,
  "title": "meeting",
  "startAt": "2026-01-02T10:00:00+09:00",
  "endAt": "2026-01-02T10:30:00+09:00",
  "status": "RESERVED",
  "createdAt": "2026-01-01T10:30:00+09:00",
  "updatedAt": "2026-01-01T10:30:00+09:00"
}
```

**エラー**

| ステータス | 条件                     | code |
| ---------- | ------------------------ | ---- |
| 401        | トークンなし・無効       |      |
| 404        | 対象リソースが存在しない |      |

### 15. PATCH /api/reservations/{id}（予約キャンセル）

- 認証：必要（ADMIN, USER）
- 説明：`status` は `CANCELED` のみ受け付ける

**リクエスト**

```json
{
  "status": "CANCELED"
}
```

**レスポンス 200 OK**

```json
{
  "id": 1,
  "userId": 1,
  "roomId": 1,
  "title": "meeting",
  "startAt": "2026-01-02T10:00:00+09:00",
  "endAt": "2026-01-02T10:30:00+09:00",
  "status": "CANCELED",
  "createdAt": "2026-01-01T10:00:00+09:00",
  "updatedAt": "2026-01-02T10:00:00+09:00"
}
```

**エラー**

| ステータス | 条件                        | code                           |
| ---------- | --------------------------- | ------------------------------ |
| 400        | `status` が `CANCELED` 以外 | VALIDATION_ERROR               |
| 401        | トークンなし・無効          | UNAUTHENTICATED                |
| 403        | 他人の予約                  | ACCESS_DENIED                  |
| 404        | 予約が存在しない            | RESERVATION_NOT_FOUND          |
| 409        | 開始の24時間前を過ぎている  | RESERVATION_CANCEL_NOT_ALLOWED |
| 409        | 既にキャンセル済み          | RESERVATION_ALREADY_CANCELED   |
