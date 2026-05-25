# KurPay API Contract

## Правило владения зонами

Backend принадлежит владельцу проекта. Codex не должен менять backend-файлы, если владелец прямо не попросил сделать конкретную правку в backend.

Codex может работать с frontend и документацией. Изменения API сначала согласуются через этот контракт, чтобы frontend мог адаптироваться без скрытого переписывания backend-логики.

Base URL: `/api/v1`

Фронт отправляет запросы с `credentials: "include"`. Авторизация через `HttpOnly Secure SameSite` cookie:

- `access_token` - короткий срок жизни.
- `refresh_token` - длинный срок жизни.

Общий формат ответа:

```json
{
  "data": {},
  "meta": {},
  "error": null
}
```

Общий формат ошибки:

```json
{
  "data": null,
  "meta": {},
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Некорректные данные"
  }
}
```

## Auth API

### POST `/auth/register`

Создает пользователя и устанавливает `access_token` / `refresh_token` в cookie.

Body:

```json
{
  "username": "kur_user",
  "password": "strong_password"
}
```

Response `201 Created`:

```json
{
  "data": {
    "user": {
      "id": "usr_01",
      "username": "kur_user",
      "role": "user",
      "balance": 0,
      "rating": 0,
      "salesCount": 0,
      "createdAt": "2026-05-23T12:00:00.000Z"
    }
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `400 VALIDATION_ERROR`
- `409 USERNAME_ALREADY_EXISTS`

### POST `/auth/login`

Входит по username и паролю, устанавливает токены в cookie.

Body:

```json
{
  "username": "kur_user",
  "password": "strong_password"
}
```

Response `200 OK`: такой же, как `/auth/register`.

Ошибки:

- `401 INVALID_CREDENTIALS`
- `423 USER_BLOCKED`

### POST `/auth/refresh`

Обновляет `access_token` по `refresh_token` из cookie.

Response `200 OK`:

```json
{
  "data": {
    "ok": true
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `401 REFRESH_TOKEN_EXPIRED`
- `401 REFRESH_TOKEN_INVALID`

### POST `/auth/logout`

Очищает refresh-сессию и cookie.

Response `204 No Content`

### GET `/auth/me`

Возвращает текущего пользователя.

Response `200 OK`:

```json
{
  "data": {
    "id": "usr_01",
    "username": "kur_user",
    "role": "user",
    "balance": 1250,
    "rating": 4.8,
    "salesCount": 12,
    "isBlocked": false
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `401 UNAUTHORIZED`

## Catalog API

### GET `/apps`

Возвращает игры и приложения для главной витрины.

Query:

- `search?: string`
- `limit?: number`

Response `200 OK`:

```json
{
  "data": {
    "items": [
      {
        "id": "genshin",
        "name": "Genshin Impact",
        "tag": "донат, примогемы, аккаунты",
        "coverColor": "#31b6a5",
        "offersCount": 238
      }
    ]
  },
  "meta": {},
  "error": null
}
```

### GET `/apps/{appId}`

Детальная страница игры/приложения.

Response `200 OK`:

```json
{
  "data": {
    "id": "genshin",
    "name": "Genshin Impact",
    "tag": "донат, примогемы, аккаунты",
    "categories": ["accounts", "donate", "services", "other"]
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `404 APP_NOT_FOUND`

## Offers API

### GET `/offers`

Список услуг для главной и страниц приложений.

Query:

- `appId?: string`
- `category?: "accounts" | "donate" | "services" | "other"`
- `search?: string`
- `sort?: "new" | "price_asc" | "price_desc" | "rating" | "popular"`
- `page?: number`
- `limit?: number`

Response `200 OK`:

```json
{
  "data": {
    "items": [
      {
        "id": "of_101",
        "appId": "genshin",
        "category": "donate",
        "title": "1980 кристаллов сотворения",
        "description": "Пополнение через UID.",
        "price": 1190,
        "currency": "RUB",
        "stock": 12,
        "delivery": "5-15 минут",
        "status": "active",
        "seller": {
          "id": "usr_02",
          "username": "MoonSeller",
          "rating": 4.9,
          "salesCount": 328
        },
        "createdAt": "2026-05-23T12:00:00.000Z"
      }
    ],
    "page": 1,
    "totalPages": 8,
    "totalItems": 120
  },
  "meta": {},
  "error": null
}
```

### GET `/offers/{offerId}`

Карточка услуги.

Response `200 OK`:

```json
{
  "data": {
    "id": "of_101",
    "appId": "genshin",
    "category": "donate",
    "title": "1980 кристаллов сотворения",
    "description": "Пополнение через UID.",
    "price": 1190,
    "currency": "RUB",
    "stock": 12,
    "delivery": "5-15 минут",
    "seller": {
      "id": "usr_02",
      "username": "MoonSeller",
      "rating": 4.9,
      "salesCount": 328
    }
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `404 OFFER_NOT_FOUND`

### POST `/offers`

Создает услугу продавца. Требует авторизацию.

Body:

```json
{
  "appId": "genshin",
  "category": "donate",
  "title": "1980 кристаллов сотворения",
  "description": "Пополнение через UID.",
  "secretData": "После оплаты попросить UID и сервер.",
  "price": 1190,
  "stock": 12,
  "delivery": "5-15 минут"
}
```

Response `201 Created`:

```json
{
  "data": {
    "id": "of_101",
    "status": "active"
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `401 UNAUTHORIZED`
- `400 VALIDATION_ERROR`
- `404 APP_NOT_FOUND`

### PATCH `/offers/{offerId}`

Редактирует свою услугу.

Body: любые поля из `POST /offers`.

Response `200 OK`:

```json
{
  "data": {
    "id": "of_101",
    "appId": "genshin",
    "category": "donate",
    "title": "1980 кристаллов сотворения",
    "description": "Пополнение через UID.",
    "price": 1190,
    "currency": "RUB",
    "stock": 12,
    "delivery": "5-15 минут",
    "status": "active"
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `401 UNAUTHORIZED`
- `403 OFFER_FORBIDDEN`
- `404 OFFER_NOT_FOUND`

### DELETE `/offers/{offerId}`

Скрывает или удаляет свою услугу.

Response `204 No Content`

## Balance API

### GET `/balance`

Возвращает баланс текущего пользователя.

Response `200 OK`:

```json
{
  "data": {
    "balance": 1250,
    "currency": "RUB"
  },
  "meta": {},
  "error": null
}
```

### POST `/balance/top-up/test`

Тестовое пополнение без реальной оплаты.

Body:

```json
{
  "amount": 500
}
```

Response `200 OK`:

```json
{
  "data": {
    "balance": 1750,
    "transactionId": "txn_01"
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `401 UNAUTHORIZED`
- `400 INVALID_AMOUNT`

### GET `/balance/transactions`

История операций баланса.

Query:

- `page?: number`
- `limit?: number`

Response `200 OK`:

```json
{
  "data": {
    "items": [
      {
        "id": "txn_01",
        "type": "top_up",
        "title": "Пополнение",
        "amount": 500,
        "currency": "RUB",
        "createdAt": "2026-05-23T12:00:00.000Z"
      }
    ],
    "page": 1,
    "totalPages": 1,
    "totalItems": 1
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `401 UNAUTHORIZED`

## Orders API

### POST `/orders`

Покупает услугу: списывает баланс, создает сделку и чат.

Body:

```json
{
  "offerId": "of_101"
}
```

Response `201 Created`:

```json
{
  "data": {
    "order": {
      "id": "ord_01",
      "offerId": "of_101",
      "buyerId": "usr_01",
      "sellerId": "usr_02",
      "status": "paid",
      "price": 1190,
      "chatId": "chat_01",
      "createdAt": "2026-05-23T12:00:00.000Z",
      "offer": {
        "id": "of_101",
        "title": "1980 кристаллов сотворения",
        "appId": "genshin",
        "category": "donate"
      }
    }
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `401 UNAUTHORIZED`
- `402 INSUFFICIENT_BALANCE`
- `404 OFFER_NOT_FOUND`
- `409 OFFER_OUT_OF_STOCK`
- `409 CANNOT_BUY_OWN_OFFER`

### GET `/orders/my`

Мои покупки и продажи.

Query:

- `type?: "buy" | "sell"`
- `status?: "paid" | "in_progress" | "done" | "dispute" | "cancelled"`

Response `200 OK`:

```json
{
  "data": {
    "items": [
      {
        "id": "ord_01",
        "offerId": "of_101",
        "buyerId": "usr_01",
        "sellerId": "usr_02",
        "status": "paid",
        "price": 1190,
        "chatId": "chat_01",
        "createdAt": "2026-05-23T12:00:00.000Z",
        "offer": {
          "id": "of_101",
          "title": "1980 кристаллов сотворения",
          "appId": "genshin",
          "category": "donate"
        }
      }
    ]
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `401 UNAUTHORIZED`

### GET `/orders/{orderId}`

Детали сделки.

Response `200 OK`:

```json
{
  "data": {
    "id": "ord_01",
    "offerId": "of_101",
    "buyerId": "usr_01",
    "sellerId": "usr_02",
    "status": "paid",
    "price": 1190,
    "chatId": "chat_01",
    "createdAt": "2026-05-23T12:00:00.000Z",
    "offer": {
      "id": "of_101",
      "title": "1980 кристаллов сотворения",
      "appId": "genshin",
      "category": "donate",
      "description": "Пополнение через UID."
    },
    "buyer": {
      "id": "usr_01",
      "username": "kur_user"
    },
    "seller": {
      "id": "usr_02",
      "username": "MoonSeller"
    }
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `401 UNAUTHORIZED`
- `403 ORDER_FORBIDDEN`
- `404 ORDER_NOT_FOUND`

### POST `/orders/{orderId}/complete`

Покупатель подтверждает выполнение. Деньги можно переводить продавцу.

Response `200 OK`:

```json
{
  "data": {
    "id": "ord_01",
    "status": "done"
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `401 UNAUTHORIZED`
- `403 ORDER_FORBIDDEN`
- `404 ORDER_NOT_FOUND`
- `409 ORDER_STATUS_INVALID`

### POST `/orders/{orderId}/dispute`

Открывает спор по сделке.

Body:

```json
{
  "reason": "Продавец не отвечает"
}
```

Response `200 OK`:

```json
{
  "data": {
    "id": "ord_01",
    "status": "dispute"
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `401 UNAUTHORIZED`
- `403 ORDER_FORBIDDEN`
- `404 ORDER_NOT_FOUND`
- `409 ORDER_STATUS_INVALID`

## Chat API

### GET `/orders/{orderId}/messages`

Сообщения чата сделки.

Query:

- `beforeId?: string`
- `limit?: number`

Response `200 OK`:

```json
{
  "data": {
    "items": [
      {
        "id": "msg_01",
        "orderId": "ord_01",
        "senderId": "usr_01",
        "type": "text",
        "text": "Здравствуйте",
        "createdAt": "2026-05-23T12:00:00.000Z"
      }
    ]
  },
  "meta": {},
  "error": null
}
```

### POST `/orders/{orderId}/messages`

Отправляет сообщение в чат сделки.

Body:

```json
{
  "text": "Мой UID 123456789"
}
```

Response `201 Created`:

```json
{
  "data": {
    "id": "msg_02",
    "orderId": "ord_01",
    "senderId": "usr_01",
    "type": "text",
    "text": "Мой UID 123456789",
    "createdAt": "2026-05-23T12:01:00.000Z"
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `401 UNAUTHORIZED`
- `403 ORDER_FORBIDDEN`
- `404 ORDER_NOT_FOUND`

### WS `/ws/orders/{orderId}?token=...`

WebSocket сделки. Если токены в cookie корректно работают для WS, query token не нужен.

События:

```json
{ "type": "message", "text": "Привет" }
{ "type": "typing", "isTyping": true }
{ "type": "order_status", "status": "dispute" }
```

## Favorites API

### GET `/favorites`

Возвращает избранные услуги.

Response `200 OK`:

```json
{
  "data": {
    "items": [
      {
        "id": "of_101",
        "appId": "genshin",
        "category": "donate",
        "title": "1980 кристаллов сотворения",
        "price": 1190,
        "currency": "RUB",
        "seller": {
          "id": "usr_02",
          "username": "MoonSeller"
        }
      }
    ]
  },
  "meta": {},
  "error": null
}
```

### POST `/favorites/{offerId}`

Добавляет услугу в избранное.

Response `201 Created`:

```json
{
  "data": {
    "offerId": "of_101",
    "isFavorite": true
  },
  "meta": {},
  "error": null
}
```

### DELETE `/favorites/{offerId}`

Удаляет услугу из избранного.

Response `204 No Content`

## Reviews API

### POST `/orders/{orderId}/review`

Оставляет отзыв после завершенной сделки.

Body:

```json
{
  "rating": 5,
  "text": "Быстро и честно"
}
```

Response `201 Created`:

```json
{
  "data": {
    "id": "rev_01",
    "orderId": "ord_01",
    "sellerId": "usr_02",
    "buyerId": "usr_01",
    "rating": 5,
    "text": "Быстро и честно",
    "createdAt": "2026-05-23T12:00:00.000Z"
  },
  "meta": {},
  "error": null
}
```

Ошибки:

- `409 ORDER_NOT_COMPLETED`
- `409 REVIEW_ALREADY_EXISTS`

### GET `/users/{userId}/reviews`

Отзывы продавца.

Response `200 OK`:

```json
{
  "data": {
    "items": [
      {
        "id": "rev_01",
        "orderId": "ord_01",
        "buyer": {
          "id": "usr_01",
          "username": "kur_user"
        },
        "rating": 5,
        "text": "Быстро и честно",
        "createdAt": "2026-05-23T12:00:00.000Z"
      }
    ],
    "rating": 4.8,
    "totalItems": 12
  },
  "meta": {},
  "error": null
}
```

## Notifications API

### GET `/notifications`

Уведомления пользователя.

Response `200 OK`:

```json
{
  "data": {
    "items": [
      {
        "id": "ntf_01",
        "type": "order_created",
        "title": "Новая покупка",
        "text": "Покупатель оплатил услугу",
        "isRead": false,
        "createdAt": "2026-05-23T12:00:00.000Z"
      }
    ]
  },
  "meta": {},
  "error": null
}
```

### POST `/notifications/{notificationId}/read`

Помечает уведомление прочитанным.

Response `200 OK`:

```json
{
  "data": {
    "id": "ntf_01",
    "isRead": true
  },
  "meta": {},
  "error": null
}
```

### WS `/ws/notifications`

Онлайн-уведомления: новая покупка, сообщение, спор, статус сделки.

## Reports API

### POST `/reports`

Жалоба на пользователя, услугу или сделку.

Body:

```json
{
  "targetType": "order",
  "targetId": "ord_01",
  "reason": "Продавец не выдал товар"
}
```

Response `201 Created`:

```json
{
  "data": {
    "id": "rp_01",
    "status": "new"
  },
  "meta": {},
  "error": null
}
```

## Admin API

Все роуты требуют `role = admin`.

### GET `/admin/stats`

Response `200 OK`:

```json
{
  "data": {
    "usersCount": 2431,
    "offersCount": 883,
    "ordersCount": 1284,
    "reportsCount": 12,
    "turnover": 560000
  },
  "meta": {},
  "error": null
}
```

### GET `/admin/users`

Query:

- `search?: string`
- `status?: "active" | "blocked"`
- `page?: number`

Response `200 OK`:

```json
{
  "data": {
    "items": [
      {
        "id": "usr_01",
        "username": "kur_user",
        "role": "user",
        "balance": 1250,
        "isBlocked": false,
        "createdAt": "2026-05-23T12:00:00.000Z"
      }
    ],
    "page": 1,
    "totalPages": 1,
    "totalItems": 1
  },
  "meta": {},
  "error": null
}
```

### PATCH `/admin/users/{userId}/block`

Body:

```json
{
  "reason": "Мошенничество"
}
```

Response `200 OK`:

```json
{
  "data": {
    "id": "usr_01",
    "isBlocked": true
  },
  "meta": {},
  "error": null
}
```

### PATCH `/admin/users/{userId}/unblock`

Разблокирует пользователя.

Response `200 OK`:

```json
{
  "data": {
    "id": "usr_01",
    "isBlocked": false
  },
  "meta": {},
  "error": null
}
```

### GET `/admin/offers`

Список всех услуг для модерации.

Response `200 OK`:

```json
{
  "data": {
    "items": [
      {
        "id": "of_101",
        "title": "1980 кристаллов сотворения",
        "appId": "genshin",
        "category": "donate",
        "price": 1190,
        "status": "active",
        "sellerId": "usr_02",
        "createdAt": "2026-05-23T12:00:00.000Z"
      }
    ]
  },
  "meta": {},
  "error": null
}
```

### PATCH `/admin/offers/{offerId}/hide`

Скрывает услугу.

Response `200 OK`:

```json
{
  "data": {
    "id": "of_101",
    "status": "hidden"
  },
  "meta": {},
  "error": null
}
```

### GET `/admin/orders`

Список сделок.

Response `200 OK`:

```json
{
  "data": {
    "items": [
      {
        "id": "ord_01",
        "offerId": "of_101",
        "buyerId": "usr_01",
        "sellerId": "usr_02",
        "status": "paid",
        "price": 1190,
        "chatId": "chat_01",
        "createdAt": "2026-05-23T12:00:00.000Z"
      }
    ]
  },
  "meta": {},
  "error": null
}
```

### GET `/admin/orders/{orderId}/messages`

Просмотр чата сделки при споре.

Response `200 OK`: такой же, как `GET /orders/{orderId}/messages`.

### GET `/admin/reports`

Список жалоб и споров.

Query:

- `status?: "new" | "review" | "closed"`

Response `200 OK`:

```json
{
  "data": {
    "items": [
      {
        "id": "rp_01",
        "targetType": "order",
        "targetId": "ord_01",
        "reason": "Продавец не выдал товар",
        "status": "new",
        "createdAt": "2026-05-23T12:00:00.000Z"
      }
    ]
  },
  "meta": {},
  "error": null
}
```

### PATCH `/admin/reports/{reportId}/resolve`

Решение жалобы.

Body:

```json
{
  "decision": "refund_buyer",
  "comment": "Продавец не предоставил товар"
}
```

Возможные `decision`:

- `refund_buyer`
- `pay_seller`
- `close_no_action`
- `block_user`

Response `200 OK`:

```json
{
  "data": {
    "id": "rp_01",
    "status": "closed",
    "decision": "refund_buyer"
  },
  "meta": {},
  "error": null
}
```

Ошибки admin API:

- `401 UNAUTHORIZED`
- `403 ADMIN_FORBIDDEN`
- `404 NOT_FOUND`
