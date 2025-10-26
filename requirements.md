# Airbnb Clone – Backend Requirement Specifications

### 1. **User Authentication**

### Overview

Handles user registration, login, and secure access control for guests, hosts, and admins

### API Endpoints

| Method | Endpoint              | Description                                 |
| ------ | --------------------- | ------------------------------------------- |
| `POST` | `/api/auth/register/` | Register a new user account                 |
| `POST` | `/api/auth/login/`    | Authenticate a user and issue JWT token     |
| `GET`  | `/api/auth/profile/`  | Retrieve current authenticated user details |

### Input Specifications

#### Registration

```json
{
  "first_name": "Aloise",
  "last_name": "Oluoch",
  "email": "aloise@example.com",
  "password": "Pass123!",
  "role": "host"
}
```


#### Login

```json
{
  "email": "aloise@example.com",
  "password": "Pass123!"
}
```


### Output Specifications

#### Registration / Login Success

```json
{
  "user_id": "uuid",
  "first_name": "Aloise",
  "role": "host",
  "token": "jwt_token_here"
}
```

#### Error Responses

- `400 Bad Request` – Invalid or missing fields.
- `409 Conflict` – Email already registered.
- `401 Unauthorized` – Incorrect credentials

#### Validation Rules

- Email must be unique and valid format.
- Password minimum 8 characters, must include uppercase, number, and symbol.
- Role must be one of: `guest`, `host`, `admin`.

#### Performance Criteria

- Token generation and verification must complete <200ms.
- Database lookups must use indexed fields (`email`).
- Concurrent user logins supported up to 500 requests/sec

### 2. **Property Management**

### Overview

Allows hosts to create, update, and manage property listings.

### API Endpoints

| Method   | Endpoint                | Description                                    |
| -------- | ----------------------- | ---------------------------------------------- |
| `POST`   | `/api/properties/`      | Create a new property listing                  |
| `GET`    | `/api/properties/`      | Retrieve list of all properties (with filters) |
| `GET`    | `/api/properties/{id}/` | Retrieve details of a single property          |
| `PUT`    | `/api/properties/{id}/` | Update property details (host only)            |
| `DELETE` | `/api/properties/{id}/` | Delete a property listing (host only)          |

### Input Example (POST)

```json
{
  "name": "Seaside Villa",
  "description": "Oceanfront villa with 3 bedrooms and pool",
  "location": "Diani Beach, Kenya",
  "price_per_night": 150.00
}
```

### Output Example

```json
{
  "property_id": "uuid",
  "host_id": "uuid",
  "name": "Seaside Villa",
  "location": "Diani Beach, Kenya",
  "price_per_night": 150.00,
  "created_at": "2025-10-26T10:00:00Z"
}
```

### Validation Rules

- `name`, `description`, `location`, and `price_per_night` are required.
- `price_per_night` must be a positive decimal.
- Only the property owner or admin can update/delete

### Performance Criteria

- Filter and search operations must return results <500ms.
- Use database indexing on `location` and `price_per_night`.
- Support pagination and caching for high-traffic endpoints

### 3. **Booking System**

### Overview

Enables guests to create, view, and manage property bookings

### API Endpoints

| Method  | Endpoint                     | Description                      |
| ------- | ---------------------------- | -------------------------------- |
| `POST`  | `/api/bookings/`             | Create a new booking             |
| `GET`   | `/api/bookings/`             | Retrieve user’s bookings         |
| `GET`   | `/api/bookings/{id}/`        | Retrieve a specific booking      |
| `PATCH` | `/api/bookings/{id}/cancel/` | Cancel a booking (guest or host) |

### Input Example (POST)

```json
{
  "property_id": "uuid",
  "start_date": "2025-11-01",
  "end_date": "2025-11-05"
}
```

### Output Example

```json
{
  "booking_id": "uuid",
  "property_id": "uuid",
  "user_id": "uuid",
  "start_date": "2025-11-01",
  "end_date": "2025-11-05",
  "total_price": 600.00,
  "status": "pending"
}
```

### Validation Rules

- Dates must be valid and `end_date` > `start_date`.
- Property must be available for the selected dates.
- Only authenticated users can book properties.
- On cancellation: booking status must change to `canceled`.

### Performance Criteria

- Booking creation and availability check must complete <300ms.
- Ensure no overlapping bookings using transaction locks or unique constraints.
- Daily cron job clears expired “pending” bookings.