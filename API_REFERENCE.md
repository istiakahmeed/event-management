# API Reference

## Overview

This document provides detailed API reference for the Event Management Platform. All server actions are located in `/lib/actions/` and are marked with `"use server"` directive.

---

## Event Actions (`lib/actions/event.actions.ts`)

### 1. getAllEvents

Retrieves all events with optional filtering and pagination.

**Signature:**

```typescript
async function getAllEvents({
  query,
  limit = 6,
  page,
  category,
}: GetAllEventsParams): Promise<{ data: IEvent[]; totalPages: number }>;
```

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | string | No | "" | Search query for event title |
| `limit` | number | No | 6 | Items per page |
| `page` | number | Yes | - | Page number (1-indexed) |
| `category` | string | No | "" | Category name filter |

**Returns:**

```typescript
{
  data: IEvent[],         // Array of events
  totalPages: number      // Total number of pages
}
```

**Error Handling:**

- Returns `{ data: [], totalPages: 0 }` on error
- Validates MongoDB connection

**Example:**

```typescript
const result = await getAllEvents({
  query: "React Workshop",
  category: "Technology",
  page: 1,
  limit: 6,
});
```

---

### 2. getEventById

Retrieves a single event by ID with populated references.

**Signature:**

```typescript
async function getEventById(eventId: string): Promise<IEvent>;
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `eventId` | string | Yes | MongoDB ObjectId of event |

**Validation:**

- Checks if `eventId` is valid MongoDB ObjectId
- Throws "Invalid event ID" if invalid
- Throws "Event not found" if not in database

**Returns:**

```typescript
{
  _id: string,
  title: string,
  description: string,
  location: string,
  imageUrl: string,
  startDateTime: Date,
  endDateTime: Date,
  price: string,
  isFree: boolean,
  url: string,
  category: { _id: string; name: string },
  organizer: { _id: string; firstName: string; lastName: string }
}
```

**Example:**

```typescript
const event = await getEventById("507f1f77bcf86cd799439011");
```

---

### 3. createEvent

Creates a new event.

**Signature:**

```typescript
async function createEvent({
  userId,
  event,
  path,
}: CreateEventParams): Promise<IEvent>;
```

**Parameters:**

```typescript
{
  userId: string,              // User ID creating the event
  event: {
    title: string,             // Event title (required)
    description: string,       // Event description
    location: string,          // Event location
    imageUrl: string,          // Event image URL
    startDateTime: Date,       // Event start date/time
    endDateTime: Date,         // Event end date/time
    categoryId: string,        // Category ID
    price: string,             // Price (empty string if free)
    isFree: boolean,           // Is event free
    url: string                // Event website URL
  },
  path: string                 // Page path to revalidate
}
```

**Validation:**

- Validates user exists
- Validates required fields
- Throws "Organizer not found" if user doesn't exist

**Returns:** Created `IEvent` object

**Side Effects:**

- Revalidates specified path
- Updates MongoDB database

**Example:**

```typescript
const newEvent = await createEvent({
  userId: "user123",
  event: {
    title: "React Workshop",
    description: "Learn React basics",
    location: "San Francisco, CA",
    imageUrl: "https://example.com/image.jpg",
    startDateTime: new Date("2024-06-15"),
    endDateTime: new Date("2024-06-16"),
    categoryId: "cat123",
    price: "50",
    isFree: false,
    url: "https://example.com",
  },
  path: "/events",
});
```

---

### 4. updateEvent

Updates an existing event (organizer only).

**Signature:**

```typescript
async function updateEvent({
  userId,
  event,
  path,
}: UpdateEventParams): Promise<IEvent>;
```

**Parameters:**

```typescript
{
  userId: string,              // Current user ID
  event: {
    _id: string,               // Event ID to update
    title: string,
    description: string,
    location: string,
    imageUrl: string,
    startDateTime: Date,
    endDateTime: Date,
    categoryId: string,
    price: string,
    isFree: boolean,
    url: string
  },
  path: string                 // Page path to revalidate
}
```

**Authorization:**

- Only event organizer can update
- Throws "Unauthorized or event not found" if user is not organizer

**Validation:**

- Checks if `event._id` is valid MongoDB ObjectId
- Verifies user is event organizer

**Returns:** Updated `IEvent` object

**Side Effects:**

- Revalidates specified path

**Example:**

```typescript
const updated = await updateEvent({
  userId: "user123",
  event: {
    _id: "event123",
    title: "Advanced React Workshop",
    // ... other fields
  },
  path: "/events/event123",
});
```

---

### 5. deleteEvent

Deletes an event (organizer only).

**Signature:**

```typescript
async function deleteEvent({ eventId, path }: DeleteEventParams): Promise<void>;
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `eventId` | string | Yes | Event ID to delete |
| `path` | string | Yes | Page path to revalidate |

**Validation:**

- Checks if `eventId` is valid MongoDB ObjectId
- Throws "Invalid event ID" if invalid

**Side Effects:**

- Deletes event from database
- Revalidates specified path

**Example:**

```typescript
await deleteEvent({
  eventId: "event123",
  path: "/events",
});
```

---

### 6. getEventsByUser

Retrieves events organized by a specific user.

**Signature:**

```typescript
async function getEventsByUser({
  userId,
  limit = 6,
  page,
}: GetEventsByUserParams): Promise<{ data: IEvent[]; totalPages: number }>;
```

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `userId` | string | Yes | - | User ID of organizer |
| `limit` | number | No | 6 | Items per page |
| `page` | number | Yes | - | Page number |

**Returns:**

```typescript
{
  data: IEvent[],
  totalPages: number
}
```

**Error Handling:**

- Returns `{ data: [], totalPages: 0 }` on error

**Example:**

```typescript
const userEvents = await getEventsByUser({
  userId: "user123",
  page: 1,
  limit: 10,
});
```

---

### 7. getRelatedEventsByCategory

Retrieves events in the same category (excluding current event).

**Signature:**

```typescript
async function getRelatedEventsByCategory({
  categoryId,
  eventId,
  limit = 3,
  page = 1,
}: GetRelatedEventsByCategoryParams): Promise<{
  data: IEvent[];
  totalPages: number;
}>;
```

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `categoryId` | string | Yes | - | Category ID to filter by |
| `eventId` | string | Yes | - | Event ID to exclude |
| `limit` | number | No | 3 | Items per page |
| `page` | number | No | 1 | Page number |

**Returns:**

```typescript
{
  data: IEvent[],
  totalPages: number
}
```

**Example:**

```typescript
const related = await getRelatedEventsByCategory({
  categoryId: "cat123",
  eventId: "event123",
  limit: 3,
  page: 1,
});
```

---

## Order Actions (`lib/actions/order.actions.ts`)

### 1. checkoutOrder

Creates a Stripe checkout session for event purchase.

**Signature:**

```typescript
async function checkoutOrder(order: CheckoutOrderParams): void;
```

**Parameters:**

```typescript
{
  eventTitle: string,        // Event title
  eventId: string,           // Event ID
  price: string,             // Event price
  isFree: boolean,           // Is event free
  buyerId: string            // Buyer's user ID
}
```

**Flow:**

1. Creates Stripe session with event metadata
2. Redirects to Stripe checkout
3. On success, redirects to `/profile`
4. On cancel, redirects to `/`

**Example:**

```typescript
await checkoutOrder({
  eventTitle: "React Workshop",
  eventId: "event123",
  price: "50",
  isFree: false,
  buyerId: "user123",
});
```

---

### 2. createOrder

Creates order record in database (called from Stripe webhook).

**Signature:**

```typescript
async function createOrder(order: CreateOrderParams): Promise<Order>;
```

**Parameters:**

```typescript
{
  stripeId: string,          // Stripe session/payment ID
  eventId: string,           // Event ID
  buyerId: string,           // Buyer user ID
  totalAmount: string,       // Total amount paid
  createdAt: Date            // Order creation date
}
```

**Returns:** Created `Order` object

**Example:**

```typescript
const order = await createOrder({
  stripeId: "cs_123abc",
  eventId: "event123",
  buyerId: "user123",
  totalAmount: "5000",
  createdAt: new Date(),
});
```

---

### 3. getOrdersByEvent

Retrieves orders for an event with optional buyer name search.

**Signature:**

```typescript
async function getOrdersByEvent({
  searchString,
  eventId,
}: GetOrdersByEventParams): Promise<Order[]>;
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `eventId` | string | Yes | Event ID |
| `searchString` | string | Yes | Search buyer name |

**Validation:**

- Checks if `eventId` is valid MongoDB ObjectId
- Throws "Invalid event ID" if invalid

**Returns:**

```typescript
[
  {
    _id: ObjectId,
    totalAmount: string,
    createdAt: Date,
    eventTitle: string,
    eventId: string,
    buyer: string, // "FirstName LastName"
  },
];
```

**Example:**

```typescript
const orders = await getOrdersByEvent({
  eventId: "event123",
  searchString: "John",
});
```

---

### 4. getOrdersByUser

Retrieves events user has purchased tickets for.

**Signature:**

```typescript
async function getOrdersByUser({
  userId,
  limit = 3,
  page,
}: GetOrdersByUserParams): Promise<{ data: IEvent[]; totalPages: number }>;
```

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `userId` | string | Yes | - | User ID |
| `limit` | number | No | 3 | Items per page |
| `page` | number | Yes | - | Page number |

**Returns:**

```typescript
{
  data: IEvent[],            // Events user has tickets for
  totalPages: number
}
```

**Example:**

```typescript
const tickets = await getOrdersByUser({
  userId: "user123",
  page: 1,
  limit: 3,
});
```

---

## User Actions (`lib/actions/user.actions.ts`)

### 1. createUser

Creates a new user in database (called from Clerk webhook).

**Signature:**

```typescript
async function createUser(user: CreateUserParams): Promise<User>;
```

**Parameters:**

```typescript
{
  clerkId: string,           // Clerk user ID (unique)
  firstName: string,         // User first name
  lastName: string,          // User last name
  username: string,          // User username (unique)
  email: string,             // User email (unique)
  photo: string              // User photo URL
}
```

**Returns:** Created `User` object

**Called By:** Clerk webhook on `user.created` event

**Example:**

```typescript
const user = await createUser({
  clerkId: "user_123",
  firstName: "John",
  lastName: "Doe",
  username: "johndoe",
  email: "john@example.com",
  photo: "https://example.com/photo.jpg",
});
```

---

### 2. getUserById

Retrieves user by ID.

**Signature:**

```typescript
async function getUserById(userId: string): Promise<User>;
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `userId` | string | Yes | User ID (Clerk ID) |

**Returns:**

```typescript
{
  _id: ObjectId,
  clerkId: string,
  email: string,
  username: string,
  firstName: string,
  lastName: string,
  photo: string
}
```

**Error Handling:**

- Throws "User not found" if not in database

**Example:**

```typescript
const user = await getUserById("user_123");
```

---

### 3. updateUser

Updates user profile information.

**Signature:**

```typescript
async function updateUser(
  clerkId: string,
  user: UpdateUserParams,
): Promise<User>;
```

**Parameters:**

```typescript
{
  clerkId: string,           // Clerk user ID
  firstName: string,         // Updated first name
  lastName: string,          // Updated last name
  username: string,          // Updated username
  photo: string              // Updated photo URL
}
```

**Returns:** Updated `User` object

**Error Handling:**

- Throws "User update failed" if update fails

**Example:**

```typescript
const updated = await updateUser("user_123", {
  firstName: "Jane",
  lastName: "Smith",
  username: "janesmith",
  photo: "https://example.com/new-photo.jpg",
});
```

---

## Category Actions (`lib/actions/category.actions.ts`)

### 1. getAllCategories

Retrieves all event categories.

**Signature:**

```typescript
async function getAllCategories(): Promise<ICategory[]>;
```

**Returns:**

```typescript
[
  {
    _id: ObjectId,
    name: string,
  },
];
```

**Example:**

```typescript
const categories = await getAllCategories();
```

---

### 2. createCategory

Creates a new category.

**Signature:**

```typescript
async function createCategory({
  categoryName,
}: CreateCategoryParams): Promise<ICategory>;
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `categoryName` | string | Yes | Category name (must be unique) |

**Returns:**

```typescript
{
  _id: ObjectId,
  name: string
}
```

**Error Handling:**

- Database enforces unique constraint on category names

**Example:**

```typescript
const category = await createCategory({
  categoryName: "Technology",
});
```

---

## Error Handling

All functions implement consistent error handling:

### Error Response Pattern

```typescript
try {
  // Operation logic
  return result;
} catch (error) {
  handleError(error);
  // Return default value if applicable
  return defaultValue;
}
```

### Error Handler

```typescript
export const handleError = (error: unknown) => {
  console.error(error);
  throw new Error(typeof error === "string" ? error : JSON.stringify(error));
};
```

---

## Webhook APIs

### Clerk Webhook (`api/webhook/clerk/route.ts`)

Handles Clerk user events:

- `user.created` - Creates user in database
- `user.updated` - Updates user information
- `user.deleted` - Removes user

### Stripe Webhook (`api/webhook/stripe/route.ts`)

Handles Stripe events:

- `checkout.session.completed` - Creates order in database

---

## Pagination

Results are paginated using offset-based pagination:

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | number | Page number (1-indexed) |
| `limit` | number | Items per page |

**Response Format:**

```typescript
{
  data: T[],
  totalPages: number
}
```

**Calculation:**

```
skipAmount = (page - 1) * limit
totalPages = Math.ceil(totalCount / limit)
```

---

## Rate Limiting

Currently, no rate limiting is implemented. Consider implementing rate limiting for production:

```typescript
import { Ratelimit } from "@upstash/ratelimit";

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, "1 h"),
});

const { success } = await ratelimit.limit(userId);
if (!success) throw new Error("Rate limit exceeded");
```

---

## Best Practices

1. **Always validate input:**
   - Check required fields
   - Validate ObjectIds
   - Sanitize strings

2. **Handle errors gracefully:**
   - Return meaningful error messages
   - Log errors for debugging
   - Return default values when appropriate

3. **Use TypeScript:**
   - Define types for all parameters
   - Use strict mode
   - Avoid `any` type

4. **Validate authorization:**
   - Check user ownership
   - Verify permissions
   - Use middleware for authentication

5. **Optimize queries:**
   - Use indexed fields
   - Populate only needed references
   - Limit results with pagination

---

## Type Definitions

All types are defined in `/types/index.ts`:

```typescript
// User types
CreateUserParams;
UpdateUserParams;

// Event types
CreateEventParams;
UpdateEventParams;
DeleteEventParams;
GetAllEventsParams;
GetEventsByUserParams;
GetRelatedEventsByCategoryParams;
Event;
IEvent;

// Order types
CheckoutOrderParams;
CreateOrderParams;
GetOrdersByEventParams;
GetOrdersByUserParams;

// Category types
CreateCategoryParams;
ICategory;

// Utility types
UrlQueryParams;
RemoveUrlQueryParams;
SearchParamProps;
```

---

**Document Version:** 1.0.0
**Last Updated:** May 22, 2026
