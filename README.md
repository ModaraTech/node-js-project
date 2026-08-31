# 🏠 Real Estate Management API

## Project Overview

Build a **Real Estate Management REST API** using **Node.js, Express, Prisma, and PostgreSQL**.

The system will allow a real estate company to manage users, properties, and customer inquiries.

The main purpose of this project is to practise:

* REST APIs
* CRUD operations
* PostgreSQL
* Prisma
* Database relationships
* Boolean fields
* Query parameters
* Authentication
* JWT
* Password hashing
* Role-based authorization
* Ownership-based permissions
* Validation
* Middleware
* Controllers and routes


---

# 1. User Roles

The system must have three roles:

### ADMIN

The administrator can:

* Manage users
* Manage roles
* Create, update and delete any property
* View all inquiries
* Update inquiry status

### AGENT

An agent can:

* Create properties
* View properties
* Update their own properties
* Delete their own properties
* View inquiries for their own properties
* Update inquiries for their own properties

An agent **must not** be able to modify or delete another agent's property.

### CUSTOMER

A customer can:

* View properties
* View individual properties
* Create inquiries
* View their own inquiries

A customer **must not** be able to view another customer's inquiries.

---

# 2. Database Models

Create the following models.

## Role

The role must be stored in a **database table**.

Do not use a Prisma enum for roles.

Fields:

```text
id
name
```

The database should contain:

```text
ADMIN
AGENT
CUSTOMER
```

---

## User

Fields:

```text
id
name
email
passwordHash
roleId
createdAt
updatedAt
```

Requirements:

* Email must be unique.
* Password must be hashed.
* A user belongs to one role.
* A role can have many users.

Relationship:

```text
Role 1 ───────── * User
```

---

## Property

Fields:

```text
id
title
description
location
price
bedrooms
bathrooms
propertyType
isAvailable
isFeatured
agentId
createdAt
updatedAt
```

Requirements:

* A property belongs to an agent.
* An agent can have many properties.
* `isAvailable` must be Boolean.
* `isFeatured` must be Boolean.

Relationship:

```text
User/Agent 1 ───────── * Property
```

Example property types:

```text
HOUSE
APARTMENT
LAND
COMMERCIAL
```

You may store these as strings unless instructed otherwise.

---

## Inquiry

Fields:

```text
id
message
status
customerId
propertyId
createdAt
updatedAt
```

Requirements:

* An inquiry belongs to a customer.
* An inquiry belongs to a property.
* A customer can create many inquiries.
* A property can have many inquiries.

Relationship:

```text
User/Customer 1 ─────── * Inquiry

Property 1 ──────────── * Inquiry
```

Possible statuses:

```text
PENDING
CONTACTED
CLOSED
```

---

# 3. Database Relationship Summary

Your database should have these relationships:

```text
Role
  │
  │ 1
  │
  │ *
  ▼
User
  │
  ├───────────────┐
  │               │
  │ 1             │ 1
  │               │
  │ *             │ *
  ▼               ▼
Property        Inquiry
  │               ▲
  │ 1             │
  │               │ *
  └───────────────┘
```

More simply:

```text
Role 1 ───────── * User

User 1 ───────── * Property

User 1 ───────── * Inquiry

Property 1 ───── * Inquiry
```

---

# 4. Authentication

Implement authentication using **JWT**.

## Register

```http
POST /api/auth/register
```

The registration endpoint should:

1. Receive the user's name, email and password.
2. Validate the input.
3. Check whether the email already exists.
4. Hash the password.
5. Find the CUSTOMER role.
6. Create the user.
7. Never return the password or password hash in the response.

New users registering normally should become:

```text
CUSTOMER
```

Do not allow someone to register themselves as ADMIN.

---

# 5. Login

```http
POST /api/auth/login
```

The login endpoint should:

1. Find the user by email.
2. Check the password.
3. Generate a JWT.
4. Include enough information in the token to identify the user.
5. Return the token.

Example response:

```json
{
  "message": "Login successful",
  "token": "your-jwt-token"
}
```

---

# 6. Authentication Middleware

Create authentication middleware.

Suggested file:

```text
middleware/auth.js
```

The middleware should:

1. Read the token from the request.
2. Verify the JWT.
3. Identify the user.
4. Attach the authenticated user's information to `req.user`.
5. Reject requests with missing or invalid tokens.

For example, protected controllers should eventually be able to access:

```js
req.user.id
```

and the user's role.

---

# 7. Role Middleware

Create separate role authorization middleware.

Suggested file:

```text
middleware/role.js
```

It should allow you to protect routes such as:

```js
requireRole("ADMIN")
```

or:

```js
requireRole("ADMIN", "AGENT")
```

A user with the wrong role should receive:

```text
403 Forbidden
```

---

# 8. Property Endpoints

## Get all properties

```http
GET /api/properties
```

**Public**

Anyone can view properties.

---

## Get one property

```http
GET /api/properties/:id
```

**Public**

---

## Create property

```http
POST /api/properties
```

**ADMIN or AGENT**

The agent ID must come from the authenticated user.

Do **not** trust an `agentId` supplied by the client.

Use the authenticated user:

```js
req.user.id
```

---

## Update property

```http
PUT /api/properties/:id
```

Allowed:

```text
ADMIN
```

or:

```text
AGENT who owns the property
```

An agent must not update another agent's property.

---

## Delete property

```http
DELETE /api/properties/:id
```

Allowed:

```text
ADMIN
```

or:

```text
AGENT who owns the property
```

---

# 9. Boolean Filtering

Properties contain:

```text
isAvailable
isFeatured
```

Implement filtering using query parameters.

Examples:

```http
GET /api/properties?available=true
```

```http
GET /api/properties?featured=true
```

```http
GET /api/properties?available=true&featured=true
```

Remember:

> Query parameters arrive as strings.

Therefore:

```text
"true"
```

needs to be converted into:

```text
true
```

before being used as a Boolean condition.

---

# 10. Property Filtering

Implement property filtering using query parameters.

### Property type

```http
GET /api/properties?type=APARTMENT
```

### Minimum price

```http
GET /api/properties?minPrice=5000000
```

### Maximum price

```http
GET /api/properties?maxPrice=15000000
```

### Combined filtering

```http
GET /api/properties?type=APARTMENT&available=true&minPrice=5000000&maxPrice=15000000
```

Use appropriate Prisma filtering operators such as:

```text
gte
lte
```

where necessary.

---

# 11. Inquiry Endpoints

## Create an inquiry

```http
POST /api/properties/:propertyId/inquiries
```

**CUSTOMER only**

Example request:

```json
{
  "message": "I would like to schedule a viewing."
}
```

The customer ID must come from:

```js
req.user.id
```

Do not trust a `customerId` sent in the request body.

---

## View my inquiries

```http
GET /api/inquiries/my
```

**CUSTOMER only**

A customer should only receive their own inquiries.

---

## View inquiries for a property

```http
GET /api/properties/:propertyId/inquiries
```

Allowed:

```text
ADMIN
```

or:

```text
AGENT who owns the property
```

---

## Update inquiry

```http
PUT /api/inquiries/:id
```

Allowed:

```text
ADMIN
```

or:

```text
AGENT who owns the related property
```

For example:

```json
{
  "status": "CONTACTED"
}
```

---

# 12. User Management

These endpoints should be **ADMIN only**.

## Get all users

```http
GET /api/users
```

## Get one user

```http
GET /api/users/:id
```

## Delete user

```http
DELETE /api/users/:id
```

## Change user role

```http
PUT /api/users/:id/role
```

Example:

```json
{
  "roleId": "role-id-here"
}
```

---

# 13. Role Endpoints

Create role endpoints.

These should be **ADMIN only**.

```http
POST /api/roles
GET /api/roles
GET /api/roles/:id
PUT /api/roles/:id
DELETE /api/roles/:id
```

Create the initial roles:

```text
ADMIN
AGENT
CUSTOMER
```

You may create them using a seed file or through the API during development.

---

# 14. Suggested Folder Structure

Use a structure similar to:

```text
real-estate-api/
│
├── controllers/
│   ├── authController.js
│   ├── propertyController.js
│   ├── inquiryController.js
│   ├── userController.js
│   └── roleController.js
│
├── routes/
│   ├── authRoutes.js
│   ├── propertyRoutes.js
│   ├── inquiryRoutes.js
│   ├── userRoutes.js
│   └── roleRoutes.js
│
├── middleware/
│   ├── auth.js
│   └── role.js
│
├── lib/
│   └── prisma.js
│
├── prisma/
│   └── schema.prisma
│
├── .env
├── .gitignore
├── app.js
├── server.js
└── package.json
```

You may add other files if your implementation requires them.

---

# 15. Environment Variables

Create a `.env` file.

Example:

```env
DATABASE_URL="your-postgresql-database-url"
JWT_SECRET="your-secret-key"
PORT=5000
```

Do **not** commit your `.env` file.

Your `.gitignore` should include:

```text
node_modules
.env
```

---

# 16. API Permission Summary

| Action                          | ADMIN | AGENT | CUSTOMER |
| ------------------------------- | ----: | ----: | -------: |
| View properties                 |     ✅ |     ✅ |        ✅ |
| View one property               |     ✅ |     ✅ |        ✅ |
| Create property                 |     ✅ |     ✅ |        ❌ |
| Update any property             |     ✅ |     ❌ |        ❌ |
| Update own property             |     ✅ |     ✅ |        ❌ |
| Delete any property             |     ✅ |     ❌ |        ❌ |
| Delete own property             |     ✅ |     ✅ |        ❌ |
| Create inquiry                  |     ❌ |     ❌ |        ✅ |
| View own inquiries              |     ❌ |     ❌ |        ✅ |
| View inquiries for own property |     ✅ |     ✅ |        ❌ |
| Manage users                    |     ✅ |     ❌ |        ❌ |
| Change user roles               |     ✅ |     ❌ |        ❌ |
| Manage roles                    |     ✅ |     ❌ |        ❌ |

---

# 17. Validation Requirements

Your API should validate important input.

Examples:

### Registration

* Name is required.
* Email is required.
* Email must be valid.
* Email must be unique.
* Password is required.
* Password must meet the required password rules.

### Property

* Title is required.
* Description is required.
* Location is required.
* Price must be valid.
* Bedrooms must be valid.
* Bathrooms must be valid.
* Property type is required.

### Inquiry

* Message is required.
* Property must exist.

Return appropriate status codes such as:

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
500 Internal Server Error
```

---

# 18. Important Authorization Rule

Do not rely only on the user's role.

For example, this is **not enough**:

```text
AGENT → allowed to update property
```

You must also check:

```text
Does this property belong to this agent?
```

The correct logic is:

```text
Is user ADMIN?
       │
    YES ──────→ Allow
       │
      NO
       ↓
Is user the property's agent?
       │
    YES ──────→ Allow
       │
      NO
       ↓
403 Forbidden
```

The same idea applies to inquiries.

---

# 19. Expected Authentication Flow

```text
REGISTER
   ↓
User created
   ↓
CUSTOMER role assigned


LOGIN
   ↓
Email + Password
   ↓
Password verified
   ↓
JWT generated
   ↓
Client receives token


PROTECTED REQUEST
   ↓
JWT
   ↓
auth middleware
   ↓
req.user
   ↓
role middleware
   ↓
ownership check if required
   ↓
controller
   ↓
database
```

---

# 20. Project Requirements Checklist

Before submitting, make sure you have completed:

### Database

* [ ] PostgreSQL database created
* [ ] Prisma configured
* [ ] Prisma migration completed
* [ ] Role model created
* [ ] User model created
* [ ] Property model created
* [ ] Inquiry model created
* [ ] All relationships working

### Authentication

* [ ] Register endpoint
* [ ] Login endpoint
* [ ] Password hashing
* [ ] JWT generation
* [ ] JWT verification
* [ ] Authentication middleware

### Authorization

* [ ] Role middleware
* [ ] ADMIN permissions
* [ ] AGENT permissions
* [ ] CUSTOMER permissions
* [ ] Property ownership checks
* [ ] Inquiry ownership/property checks

### Properties

* [ ] Create
* [ ] Get all
* [ ] Get one
* [ ] Update
* [ ] Delete
* [ ] Available filtering
* [ ] Featured filtering
* [ ] Property type filtering
* [ ] Price filtering

### Inquiries

* [ ] Create inquiry
* [ ] Get customer's own inquiries
* [ ] Agent can view inquiries for their properties
* [ ] Update inquiry status

### Users & Roles

* [ ] Get users
* [ ] Get one user
* [ ] Delete user
* [ ] Change user role
* [ ] Create roles
* [ ] Get roles
* [ ] Update roles
* [ ] Delete roles

### Code Structure

* [ ] Controllers separated from routes
* [ ] Middleware separated
* [ ] Prisma client separated
* [ ] Environment variables used
* [ ] `.env` excluded from Git
* [ ] Appropriate HTTP status codes used
* [ ] Input validation implemented
* [ ] Errors handled properly

---

# 21. Bonus Challenges

Once the required functionality is complete, you can add:

### Pagination

```http
GET /api/properties?page=1&limit=10
```

### Search

Allow users to search by title or location.

Example:

```http
GET /api/properties?search=Westlands
```

### Sorting

Examples:

```http
GET /api/properties?sort=price_asc
```

```http
GET /api/properties?sort=price_desc
```

### Combined search

Allow multiple filters to work together:

```http
GET /api/properties?search=Nairobi&type=APARTMENT&available=true&minPrice=5000000
```

---

# 22. Submission

Your repository should contain:

```text
README.md
package.json
prisma/
controllers/
routes/
middleware/
lib/
app.js
server.js
.gitignore
```

Do not commit:

```text
.env
node_modules/
```

Your API should run successfully after another developer clones the repository and follows your setup instructions.

---

## Final Goal

By the end of the project, your API should demonstrate that you understand:

```text
Express
   ↓
Routes
   ↓
Controllers
   ↓
Prisma
   ↓
PostgreSQL
   ↓
Relationships
   ↓
CRUD
   ↓
Validation
   ↓
Password Hashing
   ↓
JWT Authentication
   ↓
Authentication Middleware
   ↓
Role Authorization
   ↓
Ownership Authorization
   ↓
Boolean Filtering
   ↓
Query Parameters
```

The important part is not simply making the endpoints work.

The backend must **enforce the permissions correctly**.

A customer should not become an agent by sending `"role": "AGENT"` in a request.

An agent should not edit another agent's property by changing an `agentId`.

A customer should not be able to request another customer's inquiries by changing an ID.

**The server must determine what the authenticated user is allowed to do.**
