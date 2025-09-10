# Airbnb Clone Backend Requirements

This document outlines the **requirement specifications** for three key backend features of the Airbnb Clone project:  

1. User Authentication  
2. Property Management  
3. Booking System  

Each section includes **API endpoints, input/output specifications, validation rules, and performance criteria**.

---

## 🔑 1. User Authentication

### Functional Requirements
- Allow users (guest or host) to register, log in, and manage sessions.  
- Secure password handling and JWT-based authentication.  
- Support OAuth (Google/Facebook) as an optional feature.  

### API Endpoints

#### `POST /api/v1/auth/register`
Registers a new user.

**Input (JSON):**
```json
{
  "name": "Max Mustermann",
  "email": "max@example.com",
  "password": "SecurePass123",
  "role": "guest"
} 
```

**Validation Rules:**
name: required, min 2 chars.
email: required, must be unique, valid format.
password: required, min 8 chars, must include numbers + letters.
role: must be either guest or host.

**Output (201 CREATED):**
```json
{
  "message": "User registered successfully",
  "userId": "u12345",
  "role": "guest"
}
```

#### `POST /api/v1/auth/login`
Authenticates a user and returns a JWT token.

**Input (JSON):**
```json
{
  "email": "max@example.com",
  "password": "SecurePass123"
}

**Output (200 OK):**
{
  "token": "jwt-token-here",
  "expiresIn": 3600,
  "role": "guest"
}
```

**Error Cases:**
- Invalid credentials → 401 Unauthorized.
- Missing fields → 400 Bad Request.

### Performance Criteria
- Requests should complete within 200ms under normal load.
- Token verification must support 1000 requests/sec.

## 🏠 2. Property Management

### Functional Requirements
- Hosts can create, update, and delete property listings.
- Properties must include: title, description, location, price, amenities, images, availability.

### API Endpoints
#### `POST /api/v1/properties`
Creates a new property (Host only).

**Input (JSON):**
```json
{
  "title": "Cozy Apartment in Essen",
  "description": "2-bedroom apartment near city center",
  "location": "Essen, Germany",
  "price": 85,
  "amenities": ["WiFi", "Kitchen", "Heating"],
  "availability": {
    "startDate": "2025-09-10",
    "endDate": "2025-12-31"
  }
}
```

**Validation Rules**

- title: required, max 100 chars
- price: required, numeric, > 0
- location: required
- availability.startDate must be earlier than availability.endDate

**Output(201 Created)**
```json
{
  "message": "Property created successfully",
  "propertyId": "p56789"
}
```

#### `PUT /api/v1/properties/:id`
Updates property details.

**Input**
Same fields as creation, with partial updates allowed.

**Output(200 OK)**
```json
{
  "message": "Property updated successfully"
}
```

#### DELETE /api/v1/properties/:id
Deletes a property.

**Output (200 OK)**
```json
{
  "message": "Property deleted successfully"
}
```

#### `GET /api/v1/properties`
Fetches all properties with filters (location, price, amenities, etc.).

**Example Request**
#### `GET /api/v1/properties?location=Essen&priceMin=50&priceMax=150&amenities=WiFi`

**Output (200 OK)**
```json
[
  {
    "propertyId": "p56789",
    "title": "Cozy Apartment in Essen",
    "price": 85,
    "location": "Essen, Germany",
    "amenities": ["WiFi", "Kitchen"]
  }
]
```

**Performance Criteria**
- Search queries must respond within 300ms.
- Must support pagination for large datasets.


## 📅 3. Booking System

### Functional Requirements
- Guests can book available properties.
- Prevent double bookings with date validation.
- Track booking status: pending, confirmed, canceled, completed.

### API Endpoints

#### `POST /api/v1/bookings`
Creates a booking request.

**Input (JSON)**
```json
{
  "propertyId": "p56789",
  "guestId": "u12345",
  "checkIn": "2025-10-01",
  "checkOut": "2025-10-07"
}
```

**Validation Rules**
- checkIn must be earlier than checkOut
- Dates must not overlap with existing confirmed bookings
- propertyId and guestId must exist

**Output (201 Created)**
```json
{
  "message": "Booking created successfully",
  "bookingId": "b78901",
  "status": "pending"
}
```

#### `PUT /api/v1/bookings/:id/cancel`
Cancels a booking (Guest or Host based on policy).

**Output (200 OK)**
```json
{
  "message": "Booking canceled successfully"
}
```

#### `GET /api/v1/bookings/:id`
Fetches booking details.

**Output (200 OK)**
```json
{
  "bookingId": "b78901",
  "propertyId": "p56789",
  "guestId": "u12345",
  "status": "confirmed",
  "checkIn": "2025-10-01",
  "checkOut": "2025-10-07"
}
```

#### `GET /api/v1/bookings/user/:userId`
Fetches all bookings for a user.

**Output (200 OK)**
```json
[
  {
    "bookingId": "b78901",
    "propertyId": "p56789",
    "status": "confirmed",
    "checkIn": "2025-10-01",
    "checkOut": "2025-10-07"
  }
]
```

**Performance Criteria**
- Booking requests should complete within 250ms.
- Must handle 500 concurrent booking requests without conflicts.

