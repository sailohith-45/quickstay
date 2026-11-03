# QuickStay REST API Documentation

Base URL: `http://localhost:3000/api`

All authenticated routes require a valid Clerk Bearer JWT token in the `Authorization` header:
```text
Authorization: Bearer <clerk_token>
```

---

## 1. User Endpoints

### 1.1 Get Current User Profile
- **Method**: `GET`
- **Endpoint**: `/user/`
- **Auth**: Required
- **Response (200 OK)**:
  ```json
  {
    "success": true,
    "role": "user",
    "recentSearchedCities": ["New York", "London"]
  }
  ```

### 1.2 Store Recent Searched City
- **Method**: `POST`
- **Endpoint**: `/user/store-recent-search`
- **Auth**: Required
- **Request Body**:
  ```json
  {
    "recentSearchedCity": "Dubai"
  }
  ```
- **Response (200 OK)**:
  ```json
  {
    "success": true,
    "message": "City added"
  }
  ```

---

## 2. Hotel Endpoints

### 2.1 Register New Hotel
- **Method**: `POST`
- **Endpoint**: `/hotels/`
- **Auth**: Required
- **Request Body**:
  ```json
  {
    "name": "The Grand Palace",
    "address": "123 Ocean Drive",
    "contact": "+1 (555) 234-5678",
    "city": "Miami"
  }
  ```
- **Response (200 OK)**:
  ```json
  {
    "success": true,
    "message": "Hotel Registered Successfully"
  }
  ```

---

## 3. Room Endpoints

### 3.1 Get All Available Rooms
- **Method**: `GET`
- **Endpoint**: `/rooms/`
- **Auth**: Public
- **Response (200 OK)**:
  ```json
  {
    "success": true,
    "rooms": [
      {
        "_id": "67f7647c197ac559e4089b96",
        "hotel": {
          "_id": "67f76393197ac559e4089b72",
          "name": "Urbanza Suites",
          "address": "Main Road 123 Street",
          "city": "New York",
          "owner": {
            "image": "https://img.clerk.com/..."
          }
        },
        "roomType": "Double Bed",
        "pricePerNight": 299,
        "amenities": ["Free WiFi", "Room Service", "Pool Access"],
        "images": ["https://res.cloudinary.com/..."],
        "isAvailable": true,
        "createdAt": "2025-04-10T06:26:04.013Z"
      }
    ]
  }
  ```

### 3.2 Create Room Listing
- **Method**: `POST`
- **Endpoint**: `/rooms/`
- **Auth**: Required (Owner)
- **Content-Type**: `multipart/form-data`
- **Form Fields**:
  - `roomType`: string (e.g. "Single Bed", "Double Bed")
  - `pricePerNight`: number
  - `amenities`: JSON string array (e.g. `'["Free WiFi", "Room Service"]'`)
  - `images`: file[] (up to 5 image files)
- **Response (200 OK)**:
  ```json
  {
    "success": true,
    "message": "Room created successfully"
  }
  ```

### 3.3 Get Rooms of Authenticated Owner
- **Method**: `GET`
- **Endpoint**: `/rooms/owner`
- **Auth**: Required (Owner)
- **Response (200 OK)**:
  ```json
  {
    "success": true,
    "rooms": [ ... ]
  }
  ```

### 3.4 Toggle Room Availability
- **Method**: `POST`
- **Endpoint**: `/rooms/toggle-availability`
- **Auth**: Required (Owner)
- **Request Body**:
  ```json
  {
    "roomId": "67f7647c197ac559e4089b96"
  }
  ```
- **Response (200 OK)**:
  ```json
  {
    "success": true,
    "message": "Room availability Updated"
  }
  ```

---

## 4. Booking Endpoints

### 4.1 Check Room Availability
- **Method**: `POST`
- **Endpoint**: `/bookings/check-availability`
- **Auth**: Public
- **Request Body**:
  ```json
  {
    "room": "67f7647c197ac559e4089b96",
    "checkInDate": "2026-12-01",
    "checkOutDate": "2026-12-05"
  }
  ```
- **Response (200 OK)**:
  ```json
  {
    "success": true,
    "isAvailable": true
  }
  ```

### 4.2 Create Booking
- **Method**: `POST`
- **Endpoint**: `/bookings/book`
- **Auth**: Required
- **Request Body**:
  ```json
  {
    "room": "67f7647c197ac559e4089b96",
    "checkInDate": "2026-12-01",
    "checkOutDate": "2026-12-05",
    "guests": 2
  }
  ```
- **Response (200 OK)**:
  ```json
  {
    "success": true,
    "message": "Booking created successfully"
  }
  ```

### 4.3 Get Current User Bookings
- **Method**: `GET`
- **Endpoint**: `/bookings/user`
- **Auth**: Required
- **Response (200 OK)**:
  ```json
  {
    "success": true,
    "bookings": [ ... ]
  }
  ```

### 4.4 Get Hotel Owner Bookings & Metrics
- **Method**: `GET`
- **Endpoint**: `/bookings/hotel`
- **Auth**: Required (Owner)
- **Response (200 OK)**:
  ```json
  {
    "success": true,
    "dashboardData": {
      "totalBookings": 12,
      "totalRevenue": 4500,
      "bookings": [ ... ]
    }
  }
  ```

### 4.5 Initialize Stripe Checkout Session
- **Method**: `POST`
- **Endpoint**: `/bookings/stripe-payment`
- **Auth**: Required
- **Request Body**:
  ```json
  {
    "bookingId": "67f76839994a731e97d3b8ce"
  }
  ```
- **Response (200 OK)**:
  ```json
  {
    "success": true,
    "url": "https://checkout.stripe.com/c/pay/cs_test_..."
  }
  ```

---

## 5. Webhooks

### 5.1 Clerk Webhook Listener
- **Method**: `POST`
- **Endpoint**: `/clerk`
- **Verification**: Svix Headers (`svix-id`, `svix-timestamp`, `svix-signature`)
- **Handled Events**:
  - `user.created` -> Creates local User document
  - `user.updated` -> Updates username, email, image
  - `user.deleted` -> Removes user document

### 5.2 Stripe Webhook Listener
- **Method**: `POST`
- **Endpoint**: `/stripe`
- **Content-Type**: `application/json` (Raw stream parsed)
- **Verification**: `stripe-signature` header via `STRIPE_WEBHOOK_SECRET`
- **Handled Events**:
  - `payment_intent.succeeded` -> Marks matching booking as `{ isPaid: true, paymentMethod: "Stripe" }`
