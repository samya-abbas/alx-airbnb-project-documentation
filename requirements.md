# 📑 Technical & Functional Requirements – Airbnb Clone Backend

This document specifies API endpoints, validation rules, and performance criteria for three core backend features:

1. User Authentication  
2. Property Management  
3. Booking System  

---

## 1. 🔐 User Authentication

| Item | Requirement |
|------|-------------|
| **Endpoints** | `POST /api/v1/auth/register`  <br>`POST /api/v1/auth/login`  <br>`POST /api/v1/auth/logout` |
| **Input** | **Register** → JSON body:<br>`{ "first_name": "Samya", "last_name": "Abbas", "email": "samya@example.com", "password": "P@ssw0rd!" }` |
| **Output** | **201** Created: `{ "token": "<JWT>", "user_id": "<uuid>" }` |
| **Validation** | • `email` unique, RFC 5322 compliant  <br>• `password` min 8 chars, 1 uppercase, 1 digit  |
| **Security** | • JWT signed with HS256, expiry = 24 h  <br>• Passwords hashed with PBKDF2 + salt |
| **Performance** | • Registration ≤ 300 ms P95  <br>• Login rate‑limit: max 5 attempts / min / IP |

---

## 2. 🏡 Property Management

| Item | Requirement |
|------|-------------|
| **Endpoints** | `POST /api/v1/properties/` (host only)  <br>`GET /api/v1/properties/{id}`  <br>`PUT /api/v1/properties/{id}` (host only)  <br>`DELETE /api/v1/properties/{id}` (host only) |
| **Input** | **Create** → multipart/form‑data:<br>- `name` (string, 1–120 chars)<br>- `description` (string, 20–3 000 chars)<br>- `location` (lat, lng OR text)<br>- `price_per_night` (decimal, ≥ 10)<br>- `images[]` (JPEG/PNG ≤ 5 MB each) |
| **Output** | **201** Created: property JSON including `property_id` |
| **Validation** | • Auth header must belong to role `host`  <br>• Max 10 images / listing  |
| **Performance** | • Full‑text search response ≤ 400 ms P95  <br>• Image upload off‑loads to S3 async via Celery |

---

## 3. 📅 Booking System

| Item | Requirement |
|------|-------------|
| **Endpoints** | `POST /api/v1/bookings/`  <br>`GET /api/v1/bookings/{id}`  <br>`GET /api/v1/users/{userId}/bookings` |
| **Input** | `POST` body:<br>`{ "property_id": "<uuid>", "start_date": "2025-08-10", "end_date": "2025-08-15", "guests": 2 }` |
| **Output** | **201** Created: `{ "booking_id": "<uuid>", "status": "pending", "total_price": 375.00 }` |
| **Business Rules** | • No overlapping bookings per property  <br>• `start_date < end_date`, stay ≤ 30 days  |
| **Validation** | • Authenticated user role = guest  <br>• Property must be `active` |
| **Performance** | • Availability check query ≤ 50 ms  <br>• Booking creation ≤ 250 ms P95 |
| **Scalability** | • Use row‑level locking on `bookings` during transactional insert  |
| **Payment Flow** | 1. Create booking (status = pending)  <br>2. Redirect to `POST /payments`  <br>3. On webhook success → set status = confirmed |

---

### 📈 Non‑Functional Across All Features

| Area | Target |
|------|--------|
| **Error Format** | JSON: `{ "error": "Invalid email" }` |
| **Rate Limiting** | 100 req/min per IP on public endpoints |
| **Logging** | JSON structured logs; request ID in every log line |
| **Monitoring** | Prometheus metrics: `http_request_duration_seconds` histogram |
