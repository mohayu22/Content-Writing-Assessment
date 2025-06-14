
# API Reference: Update a Todo

This document describes how to update a todo item using the backend APIs. Both the **Node.js** and **Golang** implementations support the standard `PUT /api/todos/:id` endpoint. The **Node.js API** also includes a convenience `PATCH` endpoint for toggling completion status.

---

## 1. Endpoint Overview

| Endpoint                      | Node.js | Golang | Description                        |
|------------------------------|---------|--------|------------------------------------|
| `PUT /api/todos/:id`         | ✅ Yes  | ✅ Yes | Full update of title, desc, status |
| `PATCH /api/todos/:id/toggle`| ✅ Yes  | ❌ No  | Toggle `completed` status only     |

> 🔓 **Authentication:** Not required for these endpoints

---

## 2. `PUT /api/todos/:id` – Update a Todo

### ✅ Supported in: Node.js & Golang

- **Method**: `PUT`
- **Purpose**: Updates one or more fields (`title`, `description`, `completed`) of an existing todo item.

---

### 🔗 Path Parameters

| Name | Type    | Description                                 | Required |
|------|---------|---------------------------------------------|----------|
| `id` | integer | ID of the todo. Must be a positive integer. | ✅ Yes   |

> ❗ A non-integer or negative ID will return `400 Bad Request`.

---

### 🧾 Headers

| Header         | Value              | Description                    |
|----------------|--------------------|--------------------------------|
| `Content-Type` | `application/json` | Must be set for JSON requests. |

---


### 📝 Request Body

At least one of the following fields must be present (**Node.js**).  
**Golang** allows an empty body but returns the existing todo unchanged.

| Field         | Type         | Required | Node.js Validation               | Golang Validation               |
|---------------|--------------|----------|----------------------------------|---------------------------------|
| `title`       | string       | ❌        | min: 1, max: 255 characters       | min: 1, max: 255 characters     |
| `description` | string/null  | ❌        | max: 1000 characters              | max: 1000 characters            |
| `completed`   | boolean      | ❌        | Must be boolean                   | Must be boolean if present      |

> ⚠️ Node.js rejects requests with no valid fields.  
> ⚠️ Golang accepts empty bodies but returns the todo unchanged.


---

### 📦 Example Request

```http
PUT /api/todos/42 HTTP/1.1
Host: localhost:3000
Content-Type: application/json

{
  "title": "Buy groceries",
  "description": "Milk, eggs, bread",
  "completed": true
}
```

---

### ✅ Success Response (`200 OK`)

```json
{
  "id": 42,
  "title": "Buy groceries",
  "description": "Milk, eggs, bread",
  "completed": true,
  "created_at": "2025-06-12T10:00:00Z",
  "updated_at": "2025-06-12T10:30:00Z"
}
```

#### 📘 Response Schema

```ts
{
  id: number,
  title: string,
  description: string | null,
  completed: boolean,
  created_at: string,  // ISO 8601
  updated_at: string   // ISO 8601 (auto-managed)
}
```

---

### ❌ Error Responses

#### 400 – Bad Request (Node.js)

```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "Validation failed",
  "details": [
    {
      "field": "title",
      "message": "must NOT have fewer than 1 characters"
    }
  ]
}
```

#### 400 – Bad Request (Golang)

```json
{
  "code": 400,
  "error": "title cannot be empty"
}
```

#### 404 – Not Found

```json
{
  "statusCode": 404,
  "error": "Not Found",
  "message": "Todo with ID '9999' not found"
}
```

#### 500 – Internal Server Error (Golang)

```json
{
  "code": 500,
  "error": "Internal Server Error",
  "details": "failed to update todo"
}
```

---

## 3. `PATCH /api/todos/:id/toggle` – Toggle Completion (Node.js Only)

### ✅ Supported in: Node.js only

- **Method**: `PATCH`
- **Purpose**: Toggles the `completed` boolean value (true → false or false → true)

---

### 🔗 Path Parameters

| Name | Type    | Description    | Required |
|------|---------|----------------|----------|
| `id` | integer | ID of the todo | ✅ Yes   |

---

### 🧾 Headers

| Header         | Value              | Description                    |
|----------------|--------------------|--------------------------------|
| `Content-Type` | `application/json` | Required, even for empty body. |

---

### 📝 Request Body

Must include an empty JSON object (`{}`) in the body. Sending no body at all will result in a 400 error from Fastify.


> ⚠️ Omitting the body completely results in `FST_ERR_CTP_EMPTY_JSON_BODY`.

```json
{}
```

---

### 📦 Example Request

```http
PATCH /api/todos/42/toggle HTTP/1.1
Host: localhost:3000
Content-Type: application/json

{}
```

---

### ✅ Success Response (`200 OK`)

```json
{
  "id": 42,
  "title": "Buy groceries",
  "description": "Milk, eggs, bread",
  "completed": false,
  "created_at": "2025-06-12T10:00:00Z",
  "updated_at": "2025-06-12T10:35:00Z"
}
```

---

### ❌ Error Responses

#### 400 – Empty Body

```json
{
  "statusCode": 400,
  "code": "FST_ERR_CTP_EMPTY_JSON_BODY",
  "error": "Bad Request",
  "message": "Body cannot be empty when content-type is set to 'application/json'"
}
```

#### 404 – Todo Not Found

```json
{
  "statusCode": 404,
  "error": "Not Found",
  "message": "Todo with ID '9999' not found"
}
```

---
## 4. Status Code Summary

| Endpoint                       | 200 OK | 400 Bad Request | 404 Not Found | 500 Internal Error |
|-------------------------------|--------|------------------|----------------|---------------------|
| `PUT /api/todos/:id`          | ✅     | ✅               | ✅             | ✅ (Golang)              |
| `PATCH /api/todos/:id/toggle` | ✅     | ✅               | ✅             | ❌                  |


---


## 5. Key Differences Between Node.js and Golang APIs

| Feature                | Node.js API                              | Golang API                              |
|------------------------|-------------------------------------------|------------------------------------------|
| `PUT /todos/:id`       | ✅ Supported                              | ✅ Supported                             |
| `PATCH /todos/:id/toggle` | ✅ Supported                           | ❌ Not available                         |
| Empty PUT Body         | ❌ Rejected with 400                      | ✅ Allowed, returns unchanged            |
| Unknown Fields         | ❌ Rejected                               | ✅ Ignored                               |
| Field Validation       | ✅ Strict                                 | ✅ Strict                                |
| ID Validation          | ✅ Must be positive                       | ✅ Must be positive                      |
| `updated_at` Timestamp | `YYYY-MM-DD HH:MM:SS` (no timezone)       | ISO 8601 format with `Z` timezone       |
| Error Format           | `{ error, details: [ { field, message, code } ] }`                  | `{ error, code }`                       |



---


### 🧠 Developer Notes

- When using the **Golang API**, toggling a todo requires:
  1. Fetching the todo via `GET /api/todos/:id`
  2. Reversing the current `completed` value
  3. Sending a `PUT` request with the new value

- When using the **Node.js API**, simply call `PATCH /api/todos/:id/toggle`.

---
