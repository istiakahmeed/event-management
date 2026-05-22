# Database Design & Architecture

Complete database design documentation for the Event Management Platform.

---

## Table of Contents

1. [Database Overview](#database-overview)
2. [Collection Design](#collection-design)
3. [Indexing Strategy](#indexing-strategy)
4. [Query Patterns](#query-patterns)
5. [Data Relationships](#data-relationships)
6. [Sharding Strategy](#sharding-strategy)
7. [Backup & Recovery](#backup--recovery)
8. [Performance Tuning](#performance-tuning)

---

## Database Overview

### Database Selection: MongoDB

**Why MongoDB:**

- Document-based model fits event data naturally
- Flexible schema allows easy evolution
- Horizontal scaling with sharding
- Strong consistency (replica sets)
- Excellent tooling and community support
- Easy to get started, scales to enterprise

**Deployment:** MongoDB Atlas (Cloud)

- Multi-region replicas
- Automatic backups
- Built-in monitoring
- Managed infrastructure

---

## Collection Design

### 1. Users Collection

**Purpose:** Store user profile and authentication data

```javascript
// Schema
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["clerkId", "email", "firstName", "lastName", "photo"],
      properties: {
        _id: {
          bsonType: "objectId",
          description: "Unique identifier",
        },
        clerkId: {
          bsonType: "string",
          description: "Clerk user ID (unique)",
          pattern: "^user_.*",
        },
        email: {
          bsonType: "string",
          description: "User email (unique)",
          pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$",
        },
        username: {
          bsonType: "string",
          description: "Display name (unique)",
        },
        firstName: {
          bsonType: "string",
          description: "User first name",
          minLength: 1,
          maxLength: 50,
        },
        lastName: {
          bsonType: "string",
          description: "User last name",
          minLength: 1,
          maxLength: 50,
        },
        photo: {
          bsonType: "string",
          description: "Photo URL",
        },
        bio: {
          bsonType: "string",
          description: "User bio (optional)",
          maxLength: 500,
        },
        createdAt: {
          bsonType: "date",
          description: "Account creation timestamp",
        },
        updatedAt: {
          bsonType: "date",
          description: "Last update timestamp",
        },
        status: {
          enum: ["active", "inactive", "suspended"],
          description: "User account status",
        },
      },
    },
  },
});

// Indexes
db.users.createIndex({ clerkId: 1 }, { unique: true });
db.users.createIndex({ email: 1 }, { unique: true });
db.users.createIndex({ username: 1 }, { unique: true });
db.users.createIndex({ createdAt: -1 });
```

**Example Document:**

```javascript
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "clerkId": "user_29w83wWoNXWOq2Z8g9dqwSGm47O",
  "email": "john@example.com",
  "username": "johndoe",
  "firstName": "John",
  "lastName": "Doe",
  "photo": "https://cdn.example.com/john.jpg",
  "bio": "Event enthusiast from SF",
  "createdAt": ISODate("2024-05-20T10:00:00Z"),
  "updatedAt": ISODate("2024-05-21T15:30:00Z"),
  "status": "active"
}
```

**Growth Metrics:**

- Initial: 1,000 documents (~1MB)
- 1 Year: 100,000 documents (~100MB)
- 5 Years: 1,000,000 documents (~1GB)

---

### 2. Events Collection

**Purpose:** Store event listings and metadata

```javascript
// Schema
db.createCollection("events", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: [
        "title",
        "organizer",
        "startDateTime",
        "endDateTime",
        "imageUrl",
      ],
      properties: {
        _id: {
          bsonType: "objectId",
          description: "Unique event identifier",
        },
        title: {
          bsonType: "string",
          description: "Event title",
          minLength: 3,
          maxLength: 200,
        },
        description: {
          bsonType: "string",
          description: "Event description",
          maxLength: 5000,
        },
        location: {
          bsonType: "string",
          description: "Event location (address)",
          maxLength: 500,
        },
        imageUrl: {
          bsonType: "string",
          description: "Event banner image URL",
        },
        startDateTime: {
          bsonType: "date",
          description: "Event start time",
        },
        endDateTime: {
          bsonType: "date",
          description: "Event end time",
        },
        category: {
          bsonType: "objectId",
          description: "Reference to Category collection",
        },
        organizer: {
          bsonType: "objectId",
          description: "Reference to User collection (event creator)",
        },
        price: {
          bsonType: "string",
          description: "Event ticket price",
        },
        isFree: {
          bsonType: "bool",
          description: "Whether event is free",
        },
        url: {
          bsonType: "string",
          description: "Event website URL (optional)",
        },
        attendeeCount: {
          bsonType: "int",
          description: "Number of people attending",
          default: 0,
        },
        capacity: {
          bsonType: "int",
          description: "Maximum capacity (optional)",
        },
        status: {
          enum: ["draft", "published", "ongoing", "completed", "cancelled"],
          description: "Event status",
        },
        createdAt: {
          bsonType: "date",
          description: "Event creation timestamp",
        },
        updatedAt: {
          bsonType: "date",
          description: "Last update timestamp",
        },
        tags: {
          bsonType: "array",
          items: {
            bsonType: "string",
          },
          description: "Event tags for search",
        },
      },
    },
  },
});

// Indexes
db.events.createIndex({ organizer: 1, createdAt: -1 });
db.events.createIndex({ category: 1 });
db.events.createIndex({ startDateTime: 1 });
db.events.createIndex({ createdAt: -1 });
db.events.createIndex({ title: "text", description: "text", tags: "text" });
db.events.createIndex({ location: 1 });
db.events.createIndex({ status: 1 });
db.events.createIndex({ isFree: 1 });
```

**Example Document:**

```javascript
{
  "_id": ObjectId("507f1f77bcf86cd799439012"),
  "title": "React Workshop 2024",
  "description": "Learn React from basics to advanced...",
  "location": "San Francisco, CA",
  "imageUrl": "https://cdn.example.com/react-workshop.jpg",
  "startDateTime": ISODate("2024-06-15T09:00:00Z"),
  "endDateTime": ISODate("2024-06-15T17:00:00Z"),
  "category": ObjectId("507f1f77bcf86cd799439050"),
  "organizer": ObjectId("507f1f77bcf86cd799439011"),
  "price": "99.99",
  "isFree": false,
  "url": "https://reactworkshop.com",
  "attendeeCount": 45,
  "capacity": 100,
  "status": "published",
  "createdAt": ISODate("2024-05-20T10:00:00Z"),
  "updatedAt": ISODate("2024-05-21T15:30:00Z"),
  "tags": ["react", "javascript", "web development"]
}
```

**Growth Metrics:**

- Initial: 100 documents (~5MB)
- 1 Year: 50,000 documents (~250MB)
- 5 Years: 1,000,000 documents (~5GB)

---

### 3. Categories Collection

**Purpose:** Store event categories

```javascript
// Schema
db.createCollection("categories", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name"],
      properties: {
        _id: {
          bsonType: "objectId",
          description: "Category ID",
        },
        name: {
          bsonType: "string",
          description: "Category name (unique)",
          minLength: 3,
          maxLength: 50,
        },
        description: {
          bsonType: "string",
          description: "Category description",
          maxLength: 500,
        },
        color: {
          bsonType: "string",
          description: "Hex color code for UI",
        },
        icon: {
          bsonType: "string",
          description: "Icon URL or identifier",
        },
        eventCount: {
          bsonType: "int",
          description: "Number of events in category",
          default: 0,
        },
        createdAt: {
          bsonType: "date",
          description: "Creation timestamp",
        },
      },
    },
  },
});

// Indexes
db.categories.createIndex({ name: 1 }, { unique: true });
db.categories.createIndex({ eventCount: -1 });
```

**Example Documents:**

```javascript
[
  {
    _id: ObjectId("507f1f77bcf86cd799439050"),
    name: "Technology",
    description: "Tech conferences, workshops, and meetups",
    color: "#3B82F6",
    icon: "computer",
    eventCount: 234,
    createdAt: ISODate("2024-01-01T00:00:00Z"),
  },
  {
    _id: ObjectId("507f1f77bcf86cd799439051"),
    name: "Sports",
    description: "Sports events and tournaments",
    color: "#EF4444",
    icon: "trophy",
    eventCount: 456,
    createdAt: ISODate("2024-01-01T00:00:00Z"),
  },
];
```

---

### 4. Orders Collection

**Purpose:** Store ticket purchase records

```javascript
// Schema
db.createCollection("orders", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["stripeId", "event", "buyer", "totalAmount"],
      properties: {
        _id: {
          bsonType: "objectId",
          description: "Order ID",
        },
        stripeId: {
          bsonType: "string",
          description: "Stripe session/payment ID (unique)",
        },
        event: {
          bsonType: "objectId",
          description: "Reference to Event collection",
        },
        buyer: {
          bsonType: "objectId",
          description: "Reference to User collection (purchaser)",
        },
        totalAmount: {
          bsonType: "string",
          description: "Order total in cents",
        },
        quantity: {
          bsonType: "int",
          description: "Number of tickets",
          default: 1,
        },
        paymentStatus: {
          enum: ["pending", "completed", "failed", "refunded"],
          description: "Payment status",
        },
        refundAmount: {
          bsonType: "string",
          description: "Refund amount if applicable",
        },
        createdAt: {
          bsonType: "date",
          description: "Order creation timestamp",
        },
        paidAt: {
          bsonType: "date",
          description: "Payment completion timestamp",
        },
        metadata: {
          bsonType: "object",
          description: "Additional order data",
        },
      },
    },
  },
});

// Indexes
db.orders.createIndex({ stripeId: 1 }, { unique: true });
db.orders.createIndex({ event: 1, createdAt: -1 });
db.orders.createIndex({ buyer: 1, createdAt: -1 });
db.orders.createIndex({ paymentStatus: 1 });
db.orders.createIndex({ createdAt: -1 });
```

**Example Document:**

```javascript
{
  "_id": ObjectId("507f1f77bcf86cd799439100"),
  "stripeId": "cs_test_b1HJ8W4H2yBVz9nJ7H2y7Z4Z5X6X7",
  "event": ObjectId("507f1f77bcf86cd799439012"),
  "buyer": ObjectId("507f1f77bcf86cd799439011"),
  "totalAmount": "9999",
  "quantity": 1,
  "paymentStatus": "completed",
  "createdAt": ISODate("2024-05-22T12:30:00Z"),
  "paidAt": ISODate("2024-05-22T12:32:00Z"),
  "metadata": {
    "eventTitle": "React Workshop 2024",
    "buyerEmail": "john@example.com"
  }
}
```

**Growth Metrics:**

- Initial: 10 documents (~1MB)
- 1 Year: 100,000 documents (~50MB)
- 5 Years: 10,000,000 documents (~5GB)

---

## Indexing Strategy

### 1. Index Types

```
Single Field Indexes:
├─ Equality queries (exact match)
├─ Range queries (>, <, >=, <=)
└─ Sorting results

Compound Indexes:
├─ Multiple field queries
├─ Efficient sorting
└─ Covering queries

Text Indexes:
├─ Full-text search
├─ Case-insensitive
└─ Stemming support

Geospatial Indexes:
├─ Location-based queries (future)
└─ Proximity search
```

### 2. Index Performance Impact

```
Index Size:
├─ Users: ~50MB (100K documents)
├─ Events: ~200MB (50K documents)
├─ Orders: ~300MB (100K documents)
└─ Total: ~550MB

Query Performance:
├─ Without index: O(n) - full collection scan
├─ With index: O(log n) - B-tree traversal
└─ Improvement: 100-1000x faster
```

### 3. Index Maintenance

```
Maintenance Tasks:
├─ Monitor index size
├─ Remove unused indexes
├─ Reindex on data anomalies
└─ Profile slow queries

Rebuild Frequency:
├─ Automatic (MongoDB)
├─ Manual if needed: db.collection.reIndex()
└─ Monitor performance

Unused Indexes:
├─ Identify via aggregation
├─ Remove to save space
├─ Reduce write overhead
```

---

## Query Patterns

### 1. Common Queries

**Get all events (with pagination):**

```javascript
db.events
  .find({})
  .skip((page - 1) * limit)
  .limit(limit)
  .sort({ createdAt: -1 })
  .lean();
```

**Search events by title:**

```javascript
db.events
  .find({
    title: { $regex: query, $options: "i" },
  })
  .sort({ createdAt: -1 });
```

**Filter by category:**

```javascript
db.events
  .find({
    category: categoryId,
    status: "published",
  })
  .sort({ startDateTime: 1 });
```

**Get events by organizer:**

```javascript
db.events.find({ organizer: userId }).sort({ createdAt: -1 }).limit(10);
```

**Get related events:**

```javascript
db.events
  .find({
    category: categoryId,
    _id: { $ne: eventId },
    status: "published",
  })
  .limit(3);
```

**Get user's orders:**

```javascript
db.orders
  .find({
    buyer: userId,
    paymentStatus: "completed",
  })
  .sort({ createdAt: -1 })
  .populate("event");
```

**Get event attendees (organizer view):**

```javascript
db.orders
  .find({
    event: eventId,
    paymentStatus: "completed",
  })
  .populate("buyer", "firstName lastName email")
  .sort({ createdAt: -1 });
```

### 2. Aggregation Queries

**Event statistics:**

```javascript
db.events.aggregate([
  {
    $match: {
      organizer: userId,
      createdAt: {
        $gte: ISODate("2024-01-01"),
        $lte: ISODate("2024-12-31"),
      },
    },
  },
  {
    $group: {
      _id: "$category",
      count: { $sum: 1 },
      avgPrice: { $avg: "$price" },
    },
  },
  {
    $sort: { count: -1 },
  },
]);
```

**Revenue by month:**

```javascript
db.orders.aggregate([
  {
    $match: {
      paymentStatus: "completed",
      paidAt: {
        $gte: ISODate("2024-01-01"),
      },
    },
  },
  {
    $group: {
      _id: {
        $dateToString: {
          format: "%Y-%m",
          date: "$paidAt",
        },
      },
      total: { $sum: { $toInt: "$totalAmount" } },
      count: { $sum: 1 },
    },
  },
  {
    $sort: { _id: 1 },
  },
]);
```

---

## Data Relationships

### 1. Relationship Types

```
One-to-Many:
User (1) ──────────→ (Many) Event
         (organizer)

User (1) ──────────→ (Many) Order
        (buyer)

Category (1) ──────────→ (Many) Event

Event (1) ──────────→ (Many) Order
```

### 2. Referential Integrity

```
Options:

1. Foreign Key Approach
   └─ Store _id as reference
   └─ Application enforces integrity

2. Denormalization
   └─ Store frequently accessed data
   └─ Trade consistency for performance

3. Hybrid
   └─ Reference + denormalized subset
   └─ Best of both worlds
```

### 3. Denormalization Strategy

```
Event Collection includes:
├─ organizer: ObjectId (reference)
│   └─ Denormalize: organizer name
│
├─ category: ObjectId (reference)
│   └─ Denormalize: category name
│
└─ attendeeCount (computed)
    └─ Update on each order

Benefits:
├─ Fewer lookups
├─ Faster queries
└─ Better read performance

Drawbacks:
├─ Data inconsistency risk
├─ Update complexity
└─ Extra storage
```

---

## Sharding Strategy

### Current Architecture (No Sharding)

```
Single Database:
├─ All data in one MongoDB cluster
├─ Replicated across 3 nodes
└─ Suitable for current scale

Limits:
├─ ~200K concurrent connections
├─ ~1TB of data
├─ ~10K writes/second
```

### Future Sharding (When Needed)

```
Shard Key: { organizer: 1, createdAt: -1 }

Advantages:
├─ Distribute by organizer
├─ Good cardinality
├─ Even distribution
└─ Query parallelization

Shards:
├─ Shard 1: Organizers A-M
├─ Shard 2: Organizers N-Z
└─ Config Servers: Metadata

Query Routing:
├─ Scatter-gather for unsharded queries
├─ Targeted for sharded queries
└─ Metadata cached for performance
```

---

## Backup & Recovery

### 1. Backup Strategy

```
Backup Frequency:
├─ Continuous (MongoDB Atlas Continuous Backup)
├─ Daily snapshots
└─ Point-in-time recovery for 30 days

Backup Retention:
├─ Last 30 days: Hourly
├─ Last 90 days: Daily
└─ Older: Weekly (optional)

Storage:
├─ MongoDB Atlas regions (replicated)
├─ Geographic redundancy
└─ 99.99% durability SLA
```

### 2. Recovery Procedures

```
Scenario 1: Single Document Corruption
├─ Find backup with document intact
├─ Query document value
├─ Update in production database
└─ Time: Minutes

Scenario 2: Partial Data Loss
├─ Identify last good backup
├─ Restore to new collection
├─ Verify data integrity
├─ Merge or replace
└─ Time: 1-2 hours

Scenario 3: Complete Database Loss
├─ Restore from full backup
├─ Verify restore integrity
├─ Failover DNS
├─ Notify users
└─ Time: 1-4 hours
```

### 3. Backup Monitoring

```
Health Checks:
├─ Backup completion status
├─ Backup size trends
├─ Restore test (monthly)
└─ Recovery time estimate

Alerts:
├─ Backup failed
├─ No backup for 24 hours
├─ Backup size anomaly
└─ Long restore time
```

---

## Performance Tuning

### 1. Query Optimization

```
Slow Query Identification:
├─ Enable query profiling
├─ Identify queries > 100ms
├─ Analyze query plans
└─ Optimize with indexes

Optimization Steps:
1. Add indexes to filter fields
2. Add indexes to sort fields
3. Use covered queries if possible
4. Rewrite query if needed
5. Monitor improvement
```

### 2. Connection Pooling

```
Pool Configuration:
├─ Min pool size: 10
├─ Max pool size: 100
├─ Idle timeout: 30s
└─ Wait queue timeout: 10s

Monitoring:
├─ Connection count
├─ Idle connections
├─ Connection errors
└─ Queue wait time
```

### 3. Memory Usage

```
Working Set:
├─ Data actively accessed
├─ Should fit in RAM
├─ Improves cache hit rate

WiredTiger Cache:
├─ Default: 50% of RAM
├─ Adjustable up to 90%
├─ Automatic eviction
└─ Performance-dependent
```

### 4. Write Performance

```
Write Concerns:
├─ acknowledged: Wait for ack
├─ unacknowledged: Fire and forget
└─ majority: Replicate to majority

Bulk Operations:
├─ insertMany() vs individual inserts
├─ updateMany() for bulk updates
├─ Better throughput
└─ Atomic per document

Indexing Impact:
├─ More indexes = slower writes
├─ Balance read vs write performance
└─ Consider selective indexes
```

---

## Monitoring Queries

### 1. View Current Operations

```javascript
db.currentOp();
db.currentOp({ secs_running: { $gt: 5 } }); // Long-running
```

### 2. Index Usage Statistics

```javascript
db.events.aggregate([{ $indexStats: {} }]);
```

### 3. Collection Statistics

```javascript
db.events.stats();
db.events.aggregate([{ $collStats: { latencyHistogram: true } }]);
```

### 4. Query Profiling

```javascript
db.setProfilingLevel(1, { slowms: 100 }); // Log slow queries
db.system.profile.find().limit(5).pretty();
db.setProfilingLevel(0); // Disable
```

---

## Database Maintenance

### 1. Routine Tasks

```
Daily:
├─ Check backup completion
├─ Monitor alerts
└─ Review slow query log

Weekly:
├─ Analyze index usage
├─ Check storage growth
└─ Review replica lag

Monthly:
├─ Test backup recovery
├─ Reindex if needed
├─ Capacity planning
└─ Performance review
```

### 2. Health Checks

```javascript
// Check replica set status
rs.status();

// Check oplog window
db.oplog.rs.find().sort({ $natural: -1 }).limit(1);

// Check indexes
db.events.getIndexes();

// Check collection size
db.events.storageSize();
```

---

## Security Considerations

### 1. Database Security

```
Access Control:
├─ Role-based access control (RBAC)
├─ Application user (limited privileges)
├─ Admin user (full access)
└─ Read-only users for analytics

Network Security:
├─ MongoDB Atlas VPC peering
├─ IP whitelist
├─ TLS/SSL encryption
└─ Encrypted at rest
```

### 2. Data Protection

```
Sensitive Data:
├─ Passwords: Hashed by Clerk
├─ Payment info: Handled by Stripe
├─ Emails: Encrypted if needed
└─ PII: Data minimization

Backups:
├─ Encrypted at rest
├─ Encrypted in transit
└─ Access controlled
```

---

**Document Version:** 1.0.0  
**Last Updated:** May 22, 2026  
**Status:** ✅ Complete
