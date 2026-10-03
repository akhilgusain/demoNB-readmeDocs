# API Guide

Human-readable summary of every module/endpoint described in the shared build spec. This is **not**
a full OpenAPI spec — for the complete, always-up-to-date, interactive contract (every field,
every status code), use the live Swagger UI:

- Swagger UI: `http://localhost:8080/swagger-ui.html`
- Raw OpenAPI JSON: `http://localhost:8080/v3/api-docs`

## Conventions

- Base path: **`/api/v1`**.
- Auth: `Authorization: Bearer <accessToken>` header, except `/auth/**` (and the Swagger UI itself).
- List endpoints accept `page`, `size`, `sort` and relevant filters, and return:
  ```json
  { "content": [...], "page": 0, "size": 20, "totalElements": 42, "totalPages": 3 }
  ```
- Errors are returned by a global `@ControllerAdvice` as:
  ```json
  {
    "timestamp": "2026-10-03T07:30:00Z",
    "status": 400,
    "error": "Bad Request",
    "message": "Validation failed",
    "path": "/api/v1/complaints",
    "validationErrors": { "title": "must not be blank" }
  }
  ```
- Roles shown below are the minimum role(s) required; `SUPER_ADMIN` can generally also reach
  society-scoped `SOCIETY_ADMIN` endpoints only for administrative/support purposes if explicitly
  implemented — treat the table below as the primary intended role per endpoint.

---

## 1. Auth — `/api/v1/auth/**`

| Method | Path | Role |
|--------|------|------|
| POST | `/auth/register` | Public |
| POST | `/auth/login` | Public |
| POST | `/auth/refresh` | Public (valid refresh token) |
| POST | `/auth/forgot-password` | Public |
| POST | `/auth/reset-password` | Public (valid reset token) |
| GET  | `/auth/me` | Any authenticated user |
| POST | `/auth/logout` | Any authenticated user |

**Example — `POST /auth/login`**

Request:
```json
{ "email": "admin@greenvalley.com", "password": "Password123!" }
```
Response:
```json
{
  "accessToken": "eyJhbGciOi...",
  "refreshToken": "8f3c2e1a-...",
  "tokenType": "Bearer",
  "expiresIn": 900,
  "user": {
    "id": "04040404-0404-0404-0404-040404040402",
    "email": "admin@greenvalley.com",
    "role": "SOCIETY_ADMIN",
    "societyId": "01010101-0101-0101-0101-010101010101"
  }
}
```

## 2. Societies — `/api/v1/societies`

| Method | Path | Role |
|--------|------|------|
| GET | `/societies` | SUPER_ADMIN |
| GET | `/societies/{id}` | SUPER_ADMIN, SOCIETY_ADMIN (own) |
| POST | `/societies` | SUPER_ADMIN |
| PUT | `/societies/{id}` | SUPER_ADMIN |
| DELETE | `/societies/{id}` | SUPER_ADMIN |

**Example — `GET /societies/{id}` response**
```json
{
  "id": "01010101-0101-0101-0101-010101010101",
  "name": "Green Valley Society",
  "city": "Bengaluru",
  "state": "Karnataka",
  "pincode": "560066",
  "contactEmail": "contact@greenvalleysociety.in"
}
```

## 3. Blocks / Flats — `/api/v1/blocks`, `/api/v1/flats`

| Method | Path | Role |
|--------|------|------|
| GET | `/blocks` | Any authenticated (own society) |
| POST | `/blocks` | SOCIETY_ADMIN |
| PUT | `/blocks/{id}` | SOCIETY_ADMIN |
| DELETE | `/blocks/{id}` | SOCIETY_ADMIN |
| GET | `/flats` | Any authenticated (own society) |
| GET | `/flats/{id}` | Any authenticated (own society) |
| POST | `/flats` | SOCIETY_ADMIN |
| PUT | `/flats/{id}` | SOCIETY_ADMIN |
| DELETE | `/flats/{id}` | SOCIETY_ADMIN |

**Example — `GET /flats?blockId=...` response (page)**
```json
{
  "content": [
    { "id": "03030303-0303-0303-0303-030303030301", "flatNumber": "A-101", "floor": 1, "flatType": "2BHK", "status": "OCCUPIED" }
  ],
  "page": 0, "size": 20, "totalElements": 6, "totalPages": 1
}
```

## 4. Residents — `/api/v1/residents`

| Method | Path | Role |
|--------|------|------|
| GET | `/residents` | SOCIETY_ADMIN |
| GET | `/residents/{id}` | SOCIETY_ADMIN, RESIDENT (self) |
| POST | `/residents` | SOCIETY_ADMIN |
| PUT | `/residents/{id}` | SOCIETY_ADMIN, RESIDENT (self, limited fields) |
| DELETE | `/residents/{id}` | SOCIETY_ADMIN |
| GET | `/residents/{id}/family-members` | SOCIETY_ADMIN, RESIDENT (self) |
| POST | `/residents/{id}/family-members` | SOCIETY_ADMIN, RESIDENT (self) |
| DELETE | `/residents/{id}/family-members/{familyMemberId}` | SOCIETY_ADMIN, RESIDENT (self) |

**Example — `POST /residents/{id}/family-members`**
```json
{ "name": "Aryan Mehta", "relation": "Son", "age": 5 }
```

## 5. Staff — `/api/v1/staff`

| Method | Path | Role |
|--------|------|------|
| GET | `/staff` | SOCIETY_ADMIN |
| POST | `/staff` | SOCIETY_ADMIN |
| PUT | `/staff/{id}` | SOCIETY_ADMIN |
| DELETE | `/staff/{id}` | SOCIETY_ADMIN |
| GET | `/staff/{id}/attendance` | SOCIETY_ADMIN, SECURITY_GUARD |
| POST | `/staff/{id}/attendance` | SOCIETY_ADMIN, SECURITY_GUARD |

**Example — `POST /staff/{id}/attendance`**
```json
{ "attendanceDate": "2026-10-03", "status": "PRESENT", "checkIn": "2026-10-03T09:00:00Z" }
```

## 6. Visitors — `/api/v1/visitors`

| Method | Path | Role |
|--------|------|------|
| GET | `/visitors` | SOCIETY_ADMIN, SECURITY_GUARD, RESIDENT (own flat) |
| POST | `/visitors` | RESIDENT (pre-approval) |
| POST | `/visitors/{id}/approve` | RESIDENT (host) |
| POST | `/visitors/{id}/reject` | RESIDENT (host) |
| POST | `/visitors/{id}/check-in` | SECURITY_GUARD |
| POST | `/visitors/{id}/check-out` | SECURITY_GUARD |

**Example — `POST /visitors` (resident pre-approval, triggers AI risk scoring)**

Request:
```json
{ "name": "Amit Chawla", "phone": "+91-9988000001", "visitorType": "PRE_APPROVED", "purpose": "Family visit", "expectedAt": "2026-10-05T17:00:00Z" }
```
Response:
```json
{
  "id": "0a0a0a0a-0a0a-0a0a-0a0a-0a0a0a0a0a01",
  "status": "PENDING",
  "qrCode": "QR-GVS-00001",
  "aiRiskScore": 12.5,
  "aiRiskLevel": "LOW"
}
```

## 7. Notices — `/api/v1/notices`

| Method | Path | Role |
|--------|------|------|
| GET | `/notices` | Any authenticated (own society) |
| POST | `/notices` | SOCIETY_ADMIN, COMMITTEE_MEMBER |
| PUT | `/notices/{id}` | SOCIETY_ADMIN, COMMITTEE_MEMBER |
| DELETE | `/notices/{id}` | SOCIETY_ADMIN, COMMITTEE_MEMBER |

**Example — `POST /notices`**
```json
{ "title": "Diwali Celebration", "content": "Join us at the clubhouse lawn.", "noticeType": "ANNOUNCEMENT" }
```

## 8. Complaints — `/api/v1/complaints`

| Method | Path | Role |
|--------|------|------|
| GET | `/complaints` | SOCIETY_ADMIN, RESIDENT (own), MAINTENANCE_STAFF (assigned) |
| POST | `/complaints` | RESIDENT |
| PUT | `/complaints/{id}` | SOCIETY_ADMIN (assign/status), MAINTENANCE_STAFF (status, if assigned) |
| GET | `/complaints/{id}/comments` | SOCIETY_ADMIN, RESIDENT (own), MAINTENANCE_STAFF (assigned) |
| POST | `/complaints/{id}/comments` | SOCIETY_ADMIN, RESIDENT (own), MAINTENANCE_STAFF (assigned) |

**Example — `POST /complaints` (triggers AI categorize + severity)**

Request:
```json
{ "title": "Water leakage in bathroom ceiling", "description": "Continuous seepage from the flat above." }
```
Response:
```json
{
  "id": "0c0c0c0c-0c0c-0c0c-0c0c-0c0c0c0c0c01",
  "category": "Plumbing",
  "aiCategory": "PLUMBING",
  "aiSeverityScore": 72.5,
  "priority": "HIGH",
  "status": "OPEN"
}
```

## 9. Billing — `/api/v1/invoices`, `/api/v1/payments`

| Method | Path | Role |
|--------|------|------|
| GET | `/invoices` | SOCIETY_ADMIN, RESIDENT (own flat) |
| POST | `/invoices` | SOCIETY_ADMIN (single or bulk-per-flat generation) |
| GET | `/invoices/{id}` | SOCIETY_ADMIN, RESIDENT (own) |
| POST | `/invoices/{id}/pay` | RESIDENT (own) |
| GET | `/reports/financial` | SOCIETY_ADMIN, COMMITTEE_MEMBER |

**Example — `POST /invoices/{id}/pay`**

Request:
```json
{ "amount": 2625.00, "paymentMethod": "UPI", "transactionRef": "TXN-GVS-100001" }
```
Response:
```json
{ "invoiceId": "0e0e0e0e-0e0e-0e0e-0e0e-0e0e0e0e0e01", "status": "PAID", "paymentId": "0f0f0f0f-0f0f-0f0f-0f0f-0f0f0f0f0f01" }
```

## 10. Amenities — `/api/v1/amenities`, `/api/v1/amenity-bookings`

| Method | Path | Role |
|--------|------|------|
| GET | `/amenities` | Any authenticated (own society) |
| POST | `/amenities` | SOCIETY_ADMIN |
| PUT | `/amenities/{id}` | SOCIETY_ADMIN |
| GET | `/amenities/{id}/availability?date=` | RESIDENT |
| POST | `/amenity-bookings` | RESIDENT |
| GET | `/amenity-bookings?mine=true` | RESIDENT |
| POST | `/amenity-bookings/{id}/cancel` | RESIDENT (own) |

**Example — `GET /amenities/{id}/availability?date=2026-10-10`**
```json
{
  "amenityId": "10101010-1010-1010-1010-101010101001",
  "date": "2026-10-10",
  "slots": [
    { "slotStart": "18:00", "slotEnd": "19:00", "available": false },
    { "slotStart": "19:00", "slotEnd": "20:00", "available": true }
  ]
}
```

## 11. Chat — `/api/v1/chat/conversations`

| Method | Path | Role |
|--------|------|------|
| GET | `/chat/conversations` | Any authenticated (own) |
| POST | `/chat/conversations` | Any authenticated |
| GET | `/chat/conversations/{id}/messages` | Any authenticated (own) |
| POST | `/chat/conversations/{id}/messages` | Any authenticated (own) — AI replies synchronously |

**Example — `POST /chat/conversations/{id}/messages`**

Request:
```json
{ "message": "How do I pay my maintenance invoice online?" }
```
Response:
```json
{
  "userMessage": { "sender": "USER", "message": "How do I pay my maintenance invoice online?" },
  "aiMessage": { "sender": "AI", "message": "You can pay your invoice from the Billing section..." }
}
```

## 12. Notifications — `/api/v1/notifications`

| Method | Path | Role |
|--------|------|------|
| GET | `/notifications` | Any authenticated (own) |
| POST | `/notifications/{id}/read` | Any authenticated (own) |

**Example — `GET /notifications` response item**
```json
{ "id": "14141414-1414-1414-1414-141414141401", "title": "Visitor Approved", "isRead": true, "channel": "IN_APP" }
```

## 13. Dashboard / Reports

| Method | Path | Role |
|--------|------|------|
| GET | `/dashboard/admin` | SOCIETY_ADMIN |
| GET | `/dashboard/resident` | RESIDENT |
| GET | `/dashboard/guard` | SECURITY_GUARD |
| GET | `/reports/complaints` | SOCIETY_ADMIN, COMMITTEE_MEMBER |
| GET | `/reports/visitors` | SOCIETY_ADMIN, COMMITTEE_MEMBER |
| GET | `/reports/financial` | SOCIETY_ADMIN, COMMITTEE_MEMBER |

**Example — `GET /dashboard/admin` response**
```json
{
  "totalFlats": 12,
  "occupiedFlats": 8,
  "openComplaints": 4,
  "pendingVisitors": 2,
  "overdueInvoiceAmount": 11970.00,
  "aiSummary": "Complaint volume is stable month-over-month; plumbing issues account for the largest share. 4 invoices are overdue totaling ₹11,970."
}
```
