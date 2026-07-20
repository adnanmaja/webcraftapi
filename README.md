# WebCraft API

Backend API for **Gamadang**, a UGM canteen & food stall ordering platform. Built with FastAPI and PostgreSQL (Supabase).

---

## Tech Stack

- **Runtime:** Python 3.11
- **Framework:** FastAPI 0.104
- **ORM:** SQLAlchemy 2.0
- **Database:** PostgreSQL (Supabase) via psycopg2
- **Auth:** JWT (python-jose) + passlib (sha256_crypt)
- **Deployment:** Vercel / Docker

---

## Setup

```bash
cd backend
pip install -r requirements.txt
```

Create `.env` in `backend/`:
```
user2=<supabase-user>
password=<supabase-password>
host2=<supabase-host>
port=<supabase-port>
dbname=<supabase-dbname>
SECRET_KEY=<your-secret-key>
```

Run:
```bash
uvicorn main:app --reload
```

Or Docker:
```bash
docker build -t webcraft-api .
docker run -p 8000:8000 webcraft-api
```

---

## Entity Relationships

```
Kantin (canteen)
  └── Warung (stall)
        ├── MenuItem (food/drink item)
        └── Order
              └── OrderItem
                    └── MenuItem

User
  ├── Warung (as owner)
  └── Order (as customer)
```

---

## API Endpoints

All endpoints are prefixed with `/api`. Base URL (dev): `http://localhost:8000`

---

### Authentication (`/api`)

#### `POST /api/login`

Authenticate user credentials and receive a JWT token.

**Request body:**
```json
{
  "email": "john@ugm.ac.id",
  "password": "secret123"
}
```

**Response `200`**:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "user_id": 1,
  "email": "john@ugm.ac.id",
  "name": "John Doe"
}
```

**Response `401`**:
```json
{
  "detail": "Incorrect email or password"
}
```

---

#### `POST /api/register`

Register a new user account.

**Request body:**
```json
{
  "name": "John Doe",
  "email": "john@ugm.ac.id",
  "password": "secret123",
  "role": "customer",
  "phone_number": "08123456789"
}
```

`role` defaults to `"customer"`. `phone_number` is optional.

**Response `200`**:
```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@ugm.ac.id",
  "role": "customer",
  "phone_number": "08123456789"
}
```

**Response `400`**:
```json
{
  "detail": "Email already registered"
}
```

---

### Users (`/api`)

#### `GET /api/users`

List all registered users.

**Response `200`**:
```json
[
  {
    "id": 1,
    "name": "John Doe",
    "email": "john@ugm.ac.id",
    "role": "customer",
    "phone_number": "08123456789"
  },
  {
    "id": 2,
    "name": "Jane Smith",
    "email": "jane@ugm.ac.id",
    "role": "seller",
    "phone_number": null
  }
]
```

---

#### `GET /api/users/{user_id}`

Get a single user by ID.

**Response `200`**:
```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@ugm.ac.id",
  "role": "customer",
  "phone_number": "08123456789"
}
```

**Response `404`**:
```json
{
  "detail": "User nggak ada"
}
```

---

#### `GET /api/users/?name=&email=`

Find a user by name **or** email (query parameter, at least one required).

| Parameter | Type   | Required |
|-----------|--------|----------|
| `name`    | string | No*      |
| `email`   | string | No*      |

\* At least one must be provided.

**Example:** `GET /api/users/?email=john@ugm.ac.id`

**Response `200`**:
```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@ugm.ac.id",
  "role": "customer",
  "phone_number": "08123456789"
}
```

**Response `400`**:
```json
{
  "detail": "Mohon berikan paramater 'email' atau 'name'"
}
```

---

#### `POST /api/users`

Create a new user (same payload as register).

**Request body:**
```json
{
  "name": "Alice",
  "email": "alice@ugm.ac.id",
  "password": "password123",
  "role": "seller",
  "phone_number": "08129876543"
}
```

**Response `200`**:
```json
{
  "id": 3,
  "name": "Alice",
  "email": "alice@ugm.ac.id",
  "role": "seller",
  "phone_number": "08129876543"
}
```

---

#### `PUT /api/users/{user_id}`

Update an existing user. All fields optional.

**Request body:**
```json
{
  "name": "Alice Updated",
  "email": "alice.new@ugm.ac.id",
  "password": "newpassword",
  "role": "customer",
  "phone_number": "0811002003"
}
```

**Response `200`**:
```json
{
  "id": 3,
  "name": "Alice Updated",
  "email": "alice.new@ugm.ac.id",
  "role": "customer",
  "phone_number": "0811002003"
}
```

---

#### `DELETE /api/users/{user_id}`

Delete a user by ID.

**Response `200`**:
```json
{
  "message": "User 3 berhasil dihapus"
}
```

---

### Kantin (Canteens) (`/api`)

#### `GET /api/kantin`

List all canteens.

**Response `200`**:
```json
[
  {
    "id": 1,
    "name": "Kantin Timur",
    "description": "Kantin di sisi timur UGM",
    "location": "Gedung Pusat UGM Lt. 1",
    "image_url": "https://example.com/kantin-timur.jpg"
  }
]
```

---

#### `GET /api/kantin/{kantin_id}`

Get a single canteen by ID.

**Response `200`**:
```json
{
  "id": 1,
  "name": "Kantin Timur",
  "description": "Kantin di sisi timur UGM",
  "location": "Gedung Pusat UGM Lt. 1",
  "image_url": "https://example.com/kantin-timur.jpg"
}
```

---

#### `GET /api/kantin/?name=&location=`

Find canteen by name **or** location. At least one parameter required.

**Response `200`**:
```json
{
  "id": 1,
  "name": "Kantin Timur",
  "description": "Kantin di sisi timur UGM",
  "location": "Gedung Pusat UGM Lt. 1",
  "image_url": "https://example.com/kantin-timur.jpg"
}
```

---

#### `POST /api/kantin`

Create a new canteen.

**Request body:**
```json
{
  "name": "Kantin Barat",
  "description": "Kantin baru di sisi barat",
  "location": "Gedung Baru Lt. 2",
  "image_url": "https://example.com/kantin-barat.jpg"
}
```

**Response `200`**:
```json
{
  "id": 2,
  "name": "Kantin Barat",
  "description": "Kantin baru di sisi barat",
  "location": "Gedung Baru Lt. 2",
  "image_url": "https://example.com/kantin-barat.jpg"
}
```

---

#### `PUT /api/kantin/{kantin_id}`

Update a canteen. All fields optional.

**Request body:**
```json
{
  "name": "Kantin Barat Renovated",
  "description": "Sudah direnovasi",
  "location": "Gedung Baru Lt. 3"
}
```

**Response `200`**:
```json
{
  "id": 2,
  "name": "Kantin Barat Renovated",
  "description": "Sudah direnovasi",
  "location": "Gedung Baru Lt. 3",
  "image_url": "https://example.com/kantin-barat.jpg"
}
```

---

#### `DELETE /api/kantin/{kantin_id}`

Delete a canteen.

**Response `200`**:
```json
{
  "message": "Kantin 2, berhasil dihapus"
}
```

---

### Warung (Stalls) (`/api`)

#### `GET /api/warung`

List all stalls.

**Response `200`**:
```json
[
  {
    "id": 1,
    "name": "Warung Sate",
    "kantin_id": 1,
    "owner_id": 2,
    "image_url": "https://example.com/warung-sate.jpg"
  }
]
```

---

#### `GET /api/warung/{warung_id}`

Get a stall by ID, **includes its menu items**.

**Response `200`**:
```json
{
  "id": 1,
  "name": "Warung Sate",
  "kantin_id": 1,
  "owner_id": 2,
  "image_url": "https://example.com/warung-sate.jpg",
  "menu_items": [
    {
      "id": 1,
      "warung_id": 1,
      "name": "Sate Ayam",
      "price": "15000.00",
      "image_url": null,
      "stock": 50
    }
  ]
}
```

---

#### `GET /api/kantin/{kantin_id}/warung`

Get all stalls in a specific canteen.

**Response `200`**:
```json
[
  {
    "id": 1,
    "name": "Warung Sate",
    "kantin_id": 1,
    "owner_id": 2,
    "image_url": "https://example.com/warung-sate.jpg",
    "menu_items": []
  }
]
```

---

#### `GET /api/user/{owner_id}/warung`

Get all stalls owned by a specific user.

**Response `200`**:
```json
[
  {
    "id": 1,
    "name": "Warung Sate",
    "kantin_id": 1,
    "owner_id": 2,
    "image_url": "https://example.com/warung-sate.jpg"
  }
]
```

---

#### `POST /api/warung`

Create a new stall. `owner_id` must reference an existing user, `kantin_id` an existing canteen.

**Request body:**
```json
{
  "name": "Warung Bakso",
  "kantin_id": 1,
  "owner_id": 2,
  "image_url": "https://example.com/warung-bakso.jpg"
}
```

**Response `200`**:
```json
{
  "id": 2,
  "name": "Warung Bakso",
  "kantin_id": 1,
  "owner_id": 2,
  "image_url": "https://example.com/warung-bakso.jpg"
}
```

---

#### `PUT /api/warung/{warung_id}`

Update a stall. All fields optional.

**Request body:**
```json
{
  "name": "Warung Bakso Sedap",
  "image_url": "https://example.com/warung-bakso-baru.jpg"
}
```

**Response `200`**:
```json
{
  "id": 2,
  "name": "Warung Bakso Sedap",
  "kantin_id": 1,
  "owner_id": 2,
  "image_url": "https://example.com/warung-bakso-baru.jpg"
}
```

---

#### `DELETE /api/warung/{warung_id}`

Delete a stall.

**Response `200`**:
```json
{
  "message": "Warung 2, berhasil dihapus"
}
```

---

### Menu Items (`/api`)

#### `GET /api/menuitem`

List all menu items across all stalls.

**Response `200`**:
```json
[
  {
    "id": 1,
    "warung_id": 1,
    "name": "Sate Ayam",
    "price": "15000.00",
    "image_url": null,
    "stock": 50
  }
]
```

---

#### `GET /api/menuitem/{menuitem_id}`

Get a single menu item by ID.

**Response `200`**:
```json
{
  "id": 1,
  "warung_id": 1,
  "name": "Sate Ayam",
  "price": "15000.00",
  "image_url": null,
  "stock": 50
}
```

---

#### `GET /api/warung/{warung_id}/menu`

Get a menu item belonging to a specific stall. _Note: returns only one item (`first()`)._

**Response `200`**:
```json
{
  "id": 1,
  "warung_id": 1,
  "name": "Sate Ayam",
  "price": "15000.00",
  "image_url": null,
  "stock": 50
}
```

---

#### `GET /api/menuitem/?name=&price=`

Find a menu item by name **or** price. At least one parameter required.

| Parameter | Type     | Required |
|-----------|----------|----------|
| `name`    | string   | No*      |
| `price`   | decimal  | No*      |

**Response `200`**:
```json
{
  "id": 1,
  "warung_id": 1,
  "name": "Sate Ayam",
  "price": "15000.00",
  "image_url": null,
  "stock": 50
}
```

---

#### `POST /api/menuitem`

Create a new menu item.

**Request body:**
```json
{
  "warung_id": 1,
  "name": "Es Teh",
  "price": 5000.00,
  "image_url": "https://example.com/esteh.jpg",
  "stock": 100
}
```

`stock` defaults to `0`. `image_url` is optional.

**Response `200`**:
```json
{
  "id": 2,
  "warung_id": 1,
  "name": "Es Teh",
  "price": "5000.00",
  "image_url": "https://example.com/esteh.jpg",
  "stock": 100
}
```

---

#### `PUT /api/menuitem/{menuitem_id}`

Update a menu item. All fields optional.

**Request body:**
```json
{
  "name": "Es Teh Manis",
  "price": 6000.00
}
```

**Response `200`**:
```json
{
  "id": 2,
  "warung_id": 1,
  "name": "Es Teh Manis",
  "price": "6000.00",
  "image_url": "https://example.com/esteh.jpg",
  "stock": 100
}
```

---

#### `DELETE /api/menuitem/{menuitem_id}`

Delete a menu item.

**Response `200`**:
```json
{
  "message": "MenuItem 2 berhasil dihapus"
}
```

---

### Orders (`/api`)

#### `GET /api/orders`

List all orders.

**Response `200`**:
```json
[
  {
    "id": 1,
    "user_id": 1,
    "warung_id": 1,
    "total_price": "20000.00",
    "payment_status": "pending",
    "created_at": "2024-01-15T10:30:00"
  }
]
```

---

#### `GET /api/orders/{order_id}`

Get a single order by ID, **includes its order items** with full menu item details.

**Response `200`**:
```json
{
  "id": 1,
  "user_id": 1,
  "warung_id": 1,
  "total_price": "20000.00",
  "payment_status": "pending",
  "created_at": "2024-01-15T10:30:00",
  "order_items": [
    {
      "id": 1,
      "order_id": 1,
      "menu_item_id": 1,
      "quantity": 2,
      "price_at_purchase": "15000.00",
      "menu_item": {
        "id": 1,
        "warung_id": 1,
        "name": "Sate Ayam",
        "price": "15000.00",
        "image_url": null,
        "stock": 48
      }
    }
  ]
}
```

---

#### `GET /api/users/{user_id}/orders`

Get all orders placed by a specific user.

**Response `200`**:
```json
[
  {
    "id": 1,
    "user_id": 1,
    "warung_id": 1,
    "total_price": "20000.00",
    "payment_status": "pending",
    "created_at": "2024-01-15T10:30:00"
  }
]
```

---

#### `GET /api/warung/{warung_id}/orders`

Get all orders placed at a specific stall.

**Response `200`**:
```json
[
  {
    "id": 1,
    "user_id": 1,
    "warung_id": 1,
    "total_price": "20000.00",
    "payment_status": "pending",
    "created_at": "2024-01-15T10:30:00"
  }
]
```

---

#### `GET /api/orders/status/{payment_status}`

Get all orders by payment status. Common values: `pending`, `paid`, `cancelled`.

**Response `200`**:
```json
[
  {
    "id": 1,
    "user_id": 1,
    "warung_id": 1,
    "total_price": "20000.00",
    "payment_status": "pending",
    "created_at": "2024-01-15T10:30:00"
  }
]
```

---

#### `POST /api/orders`

Create an order **with its items** in a single request.

**Request body:**
```json
{
  "user_id": 1,
  "warung_id": 1,
  "total_price": 30000.00,
  "payment_status": "pending",
  "order_items": [
    {
      "menu_item_id": 1,
      "quantity": 2,
      "price_at_purchase": 15000.00
    }
  ]
}
```

`payment_status` defaults to `"pending"`. `order_items` is optional (can create empty order).

**Response `200`**:
```json
{
  "id": 2,
  "user_id": 1,
  "warung_id": 1,
  "total_price": "30000.00",
  "payment_status": "pending",
  "created_at": "2024-01-15T11:00:00"
}
```

---

#### `PUT /api/orders/{order_id}`

Update order fields. Only included fields are updated.

**Request body:**
```json
{
  "total_price": 25000.00,
  "payment_status": "paid"
}
```

**Response `200`**:
```json
{
  "id": 2,
  "user_id": 1,
  "warung_id": 1,
  "total_price": "25000.00",
  "payment_status": "paid",
  "created_at": "2024-01-15T11:00:00"
}
```

---

#### `PATCH /api/orders/{order_id}/status`

Quickly update only the payment status (query parameter).

**Query parameter:** `payment_status` (string, required)

**Example:** `PATCH /api/orders/2/status?payment_status=paid`

**Response `200`**:
```json
{
  "id": 2,
  "user_id": 1,
  "warung_id": 1,
  "total_price": "25000.00",
  "payment_status": "paid",
  "created_at": "2024-01-15T11:00:00"
}
```

---

#### `DELETE /api/orders/{order_id}`

Delete an order.

**Response `200`**:
```json
{
  "message": "Order 2 berhasil dihapus"
}
```

---

### Order Items (`/api`)

#### `GET /api/order-items`

List all order items.

**Response `200`**:
```json
[
  {
    "id": 1,
    "order_id": 1,
    "menu_item_id": 1,
    "quantity": 2,
    "price_at_purchase": "15000.00"
  }
]
```

---

#### `GET /api/order-items/{order_item_id}`

Get a single order item by ID, **includes the menu item details**.

**Response `200`**:
```json
{
  "id": 1,
  "order_id": 1,
  "menu_item_id": 1,
  "quantity": 2,
  "price_at_purchase": "15000.00",
  "menu_item": {
    "id": 1,
    "warung_id": 1,
    "name": "Sate Ayam",
    "price": "15000.00",
    "image_url": null,
    "stock": 48
  }
}
```

---

#### `GET /api/orders/{order_id}/items`

Get all items belonging to a specific order.

**Response `200`**:
```json
[
  {
    "id": 1,
    "order_id": 1,
    "menu_item_id": 1,
    "quantity": 2,
    "price_at_purchase": "15000.00",
    "menu_item": {
      "id": 1,
      "warung_id": 1,
      "name": "Sate Ayam",
      "price": "15000.00",
      "image_url": null,
      "stock": 48
    }
  }
]
```

---

#### `GET /api/menu-items/{menu_item_id}/order-items`

Get all order items referencing a specific menu item.

**Response `200`**:
```json
[
  {
    "id": 1,
    "order_id": 1,
    "menu_item_id": 1,
    "quantity": 2,
    "price_at_purchase": "15000.00"
  }
]
```

---

#### `POST /api/order-items`

Create a new order item (deducts stock from the menu item).

**Request body:**
```json
{
  "order_id": 1,
  "menu_item_id": 1,
  "quantity": 1,
  "price_at_purchase": 15000.00
}
```

**Response `200`**:
```json
{
  "id": 2,
  "order_id": 1,
  "menu_item_id": 1,
  "quantity": 1,
  "price_at_purchase": "15000.00"
}
```

**Response `400`** (insufficient stock):
```json
{
  "detail": "Stok menu item tidak cukup"
}
```

---

#### `PUT /api/order-items/{order_item_id}`

Update an order item. Adjusts stock accordingly when quantity changes.

**Request body:**
```json
{
  "quantity": 3,
  "price_at_purchase": 14000.00
}
```

**Response `200`**:
```json
{
  "id": 1,
  "order_id": 1,
  "menu_item_id": 1,
  "quantity": 3,
  "price_at_purchase": "14000.00"
}
```

---

#### `DELETE /api/order-items/{order_item_id}`

Delete an order item (restores stock to the menu item).

**Response `200`**:
```json
{
  "message": "Order item 1 berhasil dihapus"
}
```

---

## Error Responses

All endpoints return errors in this format:

```json
{
  "detail": "<error message>"
}
```

Common HTTP status codes used:
| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Bad request (validation, duplicate, insufficient stock) |
| 401 | Unauthorized (login failed) |
| 404 | Resource not found |



