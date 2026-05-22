# Event Management Platform - Complete Documentation

## Table of Contents

1. [Project Overview](#project-overview)
2. [Tech Stack](#tech-stack)
3. [Features](#features)
4. [Installation & Setup](#installation--setup)
5. [Project Structure](#project-structure)
6. [Database Schema](#database-schema)
7. [API Documentation](#api-documentation)
8. [Authentication](#authentication)
9. [Payment Integration](#payment-integration)
10. [File Upload](#file-upload)
11. [Components Guide](#components-guide)
12. [Environment Variables](#environment-variables)
13. [Running the Application](#running-the-application)
14. [Deployment](#deployment)
15. [Troubleshooting](#troubleshooting)
16. [Contributing](#contributing)

---

## Project Overview

**Event Management Platform** is a full-stack web application built with Next.js 14 that allows users to:

- Create and manage events
- Browse and search for events
- Filter events by category
- Purchase tickets for events (with Stripe payment integration)
- Manage user profiles
- View event attendance history
- Manage events as an organizer

The platform is designed with scalability, security, and user experience in mind, utilizing modern web technologies and best practices.

---

## Tech Stack

### Frontend

- **Next.js 14** - React framework with App Router
- **React 18** - UI library
- **TypeScript** - Type safety
- **Tailwind CSS** - Utility-first CSS framework
- **Radix UI** - Headless UI components
- **React Hook Form** - Form state management
- **Zod** - TypeScript-first schema validation

### Backend

- **Next.js API Routes** - Serverless functions
- **Node.js** - JavaScript runtime
- **MongoDB** - NoSQL database
- **Mongoose** - MongoDB ODM

### Authentication & Security

- **Clerk** - Modern authentication and user management
- **Middleware** - Request authentication

### Payment Processing

- **Stripe** - Payment gateway integration
- **Stripe Webhooks** - Event-driven order processing

### File Management

- **UploadThing** - File upload service
- **Next.js Image** - Image optimization

### Other Tools

- **Query String** - URL parameter handling
- **React DatePicker** - Date selection component
- **Lucide React** - Icon library
- **SVix** - Webhook management

---

## Features

### 1. **User Management**

- User registration and login via Clerk
- User profile management (name, username, photo)
- Profile picture upload
- User preference settings

### 2. **Event Management**

- **Create Events**: Users can create new events with:
  - Event title, description, location
  - Event date and time (start and end)
  - Event category
  - Pricing information (free or paid)
  - Event image upload
  - Event URL

- **View Events**:
  - Browse all events on the platform
  - View event details
  - See organizer information
  - View ticket information

- **Update Events**: Organizers can edit their events
- **Delete Events**: Organizers can remove their events

### 3. **Event Discovery**

- Search events by title
- Filter events by category
- Pagination for event listings
- Related events suggestion based on category

### 4. **Ticket Purchasing**

- One-click checkout via Stripe
- Purchase confirmation
- Ticket history tracking
- Free event registration

### 5. **Event Organization**

- Organizer dashboard
- View events created
- Track ticket sales
- See attendee information

### 6. **Categories**

- Browse events by category
- Category filtering
- Pre-defined categories (Sports, Music, Entertainment, etc.)

---

## Installation & Setup

### Prerequisites

- Node.js 18+
- npm or yarn package manager
- MongoDB Atlas account
- Stripe account
- Clerk account
- UploadThing account

### Step 1: Clone the Repository

```bash
git clone https://github.com/istiakahmeed/event-management.git
cd event-management
```

### Step 2: Install Dependencies

```bash
npm install
# or
yarn install
```

### Step 3: Set Up Environment Variables

Create a `.env` file in the root directory with the following variables:

```bash
# MongoDB
MONGODB_URI=your_mongodb_connection_string

# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
CLERK_SECRET_KEY=your_clerk_secret_key
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/

# Stripe
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=your_stripe_publishable_key
STRIPE_SECRET_KEY=your_stripe_secret_key
STRIPE_WEBHOOK_SECRET=your_stripe_webhook_secret

# UploadThing
UPLOADTHING_SECRET=your_uploadthing_secret
UPLOADTHING_APP_ID=your_uploadthing_app_id

# Server Configuration
NEXT_PUBLIC_SERVER_URL=http://localhost:3000

# Webhooks
WEBHOOK_SECRET=your_webhook_secret
```

### Step 4: Start the Development Server

```bash
npm run dev
# or
yarn dev
```

The application will be available at `http://localhost:3000`

---

## Project Structure

```
event-management/
├── app/                          # Next.js App Router
│   ├── (auth)/                   # Authentication routes
│   │   ├── sign-in/              # Sign-in page
│   │   └── sign-up/              # Sign-up page
│   ├── (root)/                   # Main application routes
│   │   ├── page.tsx              # Home page with event listing
│   │   ├── events/
│   │   │   ├── [id]/             # Event detail page
│   │   │   ├── [id]/update/      # Event update page
│   │   │   └── create/           # Create event page
│   │   ├── orders/               # Orders page
│   │   ├── profile/              # User profile page
│   │   └── layout.tsx            # Root layout
│   ├── api/                      # API routes
│   │   ├── webhook/
│   │   │   ├── clerk/            # Clerk webhook handler
│   │   │   └── stripe/           # Stripe webhook handler
│   │   ├── uploadthing/          # UploadThing routes
│   │   └── route.ts
│   ├── layout.tsx                # Global layout
│   └── globals.css               # Global styles
├── components/
│   ├── shared/                   # Shared components
│   │   ├── Card.tsx              # Event card component
│   │   ├── Checkout.tsx          # Checkout component
│   │   ├── CheckoutButton.tsx    # Checkout button
│   │   ├── Collection.tsx        # Events collection display
│   │   ├── DeleteConfirmation.tsx # Confirmation dialog
│   │   ├── EventForm.tsx         # Event creation/edit form
│   │   ├── FileUploader.tsx      # File upload component
│   │   ├── Footer.tsx            # Footer component
│   │   ├── Header.tsx            # Header/Navigation
│   │   ├── Search.tsx            # Search component
│   │   ├── Pagination.tsx        # Pagination controls
│   │   ├── CategoryFilter.tsx    # Category filter component
│   │   └── ...other components
│   └── ui/                       # Radix UI components
│       ├── button.tsx
│       ├── form.tsx
│       ├── input.tsx
│       ├── select.tsx
│       ├── sheet.tsx
│       └── ...other UI components
├── lib/
│   ├── actions/                  # Server actions
│   │   ├── event.actions.ts      # Event operations
│   │   ├── user.actions.ts       # User operations
│   │   ├── order.actions.ts      # Order operations
│   │   └── category.actions.ts   # Category operations
│   ├── database/
│   │   ├── index.ts              # Database connection
│   │   └── models/               # Mongoose schemas
│   │       ├── event.model.ts
│   │       ├── user.model.ts
│   │       ├── category.model.ts
│   │       └── order.model.ts
│   ├── uploadthing.ts            # UploadThing configuration
│   ├── utils.ts                  # Utility functions
│   └── validator.ts              # Input validation
├── types/
│   └── index.ts                  # TypeScript type definitions
├── constants/
│   └── index.ts                  # Application constants
├── public/
│   └── assets/                   # Static assets
│       ├── icons/
│       └── images/
├── middleware.ts                 # Clerk middleware
├── next.config.mjs               # Next.js configuration
├── tsconfig.json                 # TypeScript configuration
├── tailwind.config.ts            # Tailwind CSS configuration
├── postcss.config.js             # PostCSS configuration
├── components.json               # Shadcn UI configuration
└── package.json                  # Project dependencies
```

---

## Database Schema

### User Collection

```typescript
{
  _id: ObjectId,
  clerkId: String (unique),
  email: String (unique),
  username: String (unique),
  firstName: String,
  lastName: String,
  photo: String
}
```

### Event Collection

```typescript
{
  _id: ObjectId,
  title: String,
  description: String,
  location: String,
  createdAt: Date,
  imageUrl: String,
  startDateTime: Date,
  endDateTime: Date,
  price: String,
  isFree: Boolean,
  url: String,
  category: ObjectId (ref: Category),
  organizer: ObjectId (ref: User)
}
```

### Category Collection

```typescript
{
  _id: ObjectId,
  name: String (unique)
}
```

### Order Collection

```typescript
{
  _id: ObjectId,
  stripeId: String,
  totalAmount: String,
  createdAt: Date,
  event: ObjectId (ref: Event),
  buyer: ObjectId (ref: User)
}
```

---

## API Documentation

### Event Actions

#### Get All Events

```typescript
getAllEvents({
  query: string, // Search query
  category: string, // Category name to filter by
  limit: number, // Items per page (default: 6)
  page: number, // Page number (1-indexed)
});
```

**Returns:** `{ data: Event[], totalPages: number }`
**Error Handling:** Returns empty array on error

#### Get Event by ID

```typescript
getEventById(eventId: string)
```

**Returns:** `Event`
**Validation:** Checks if eventId is valid MongoDB ObjectId
**Error Handling:** Throws error if not found

#### Create Event

```typescript
createEvent({
  userId: string,
  event: {
    title: string,
    description: string,
    location: string,
    imageUrl: string,
    startDateTime: Date,
    endDateTime: Date,
    categoryId: string,
    price: string,
    isFree: boolean,
    url: string,
  },
  path: string, // Page to revalidate
});
```

**Returns:** `Event`

#### Update Event

```typescript
updateEvent({
  userId: string,
  event: {
    _id: string,
    // ...all event fields
  },
  path: string,
});
```

**Returns:** `Event`
**Authorization:** Only event organizer can update

#### Delete Event

```typescript
deleteEvent({
  eventId: string,
  path: string,
});
```

**Returns:** void
**Validation:** Validates ObjectId format

#### Get Events by Organizer

```typescript
getEventsByUser({
  userId: string,
  limit?: number,
  page: number
})
```

**Returns:** `{ data: Event[], totalPages: number }`

#### Get Related Events

```typescript
getRelatedEventsByCategory({
  categoryId: string,
  eventId: string,
  limit?: number,
  page: number | string
})
```

**Returns:** `{ data: Event[], totalPages: number }`

---

### Order Actions

#### Checkout Order

```typescript
checkoutOrder({
  eventTitle: string,
  eventId: string,
  price: string,
  isFree: boolean,
  buyerId: string,
});
```

**Redirects:** To Stripe checkout session

#### Create Order

```typescript
createOrder({
  stripeId: string,
  eventId: string,
  buyerId: string,
  totalAmount: string,
  createdAt: Date,
});
```

**Returns:** `Order`

#### Get Orders by Event

```typescript
getOrdersByEvent({
  eventId: string,
  searchString: string, // Search buyer name
});
```

**Returns:** `Order[]`

#### Get Orders by User

```typescript
getOrdersByUser({
  userId: string,
  limit?: number,
  page: number
})
```

**Returns:** `Event[]` (Events user has purchased tickets for)

---

### User Actions

#### Create User

```typescript
createUser(user: {
  clerkId: string,
  firstName: string,
  lastName: string,
  username: string,
  email: string,
  photo: string
})
```

**Returns:** `User`
**Called by:** Clerk webhook on sign-up

#### Get User by ID

```typescript
getUserById(userId: string)
```

**Returns:** `User`

#### Update User

```typescript
updateUser(clerkId: string, user: {
  firstName: string,
  lastName: string,
  username: string,
  photo: string
})
```

**Returns:** `User`

---

### Category Actions

#### Get All Categories

```typescript
getAllCategories();
```

**Returns:** `Category[]`

#### Create Category

```typescript
createCategory({ categoryName: string });
```

**Returns:** `Category`

---

## Authentication

### Clerk Setup

The application uses **Clerk** for authentication and user management.

#### Key Features:

- Social authentication (Google, GitHub, etc.)
- Email/password authentication
- Multi-factor authentication
- User profile management
- Webhooks for user events

#### Public Routes:

- `/` (Home page)
- `/sign-in` (Sign-in page)
- `/sign-up` (Sign-up page)
- `/api/webhook/clerk` (Clerk webhook)
- `/api/webhook/stripe` (Stripe webhook)
- `/api/uploadthing` (File upload endpoint)

#### Protected Routes:

All other routes require authentication via Clerk middleware

#### Middleware Configuration:

```typescript
// middleware.ts
authMiddleware({
  publicRoutes: [
    "/",
    "/events/clerk",
    "/api/webhook/stripe",
    "/api/uploadthing",
    "/api/webhook/clerk",
  ],
  ignoredRoutes: ["/events/clerk", "/api/webhook/stripe", "/api/uploadthing"],
});
```

#### User Creation Workflow:

1. User signs up via Clerk
2. Clerk triggers webhook to `/api/webhook/clerk`
3. Backend creates user in MongoDB
4. User can now create events and purchase tickets

---

## Payment Integration

### Stripe Setup

The application uses **Stripe** for processing payments.

#### Payment Flow:

1. User clicks "Buy Tickets"
2. `checkoutOrder` creates Stripe session
3. User redirected to Stripe checkout
4. Payment processed
5. Stripe webhook sent to `/api/webhook/stripe`
6. Order created in database
7. User redirected to profile page

#### Webhook Handling:

Stripe webhooks are processed at `/api/webhook/stripe/route.ts`

**Handled Events:**

- `checkout.session.completed` - Creates order record

#### Environment Variables Required:

```
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY
STRIPE_SECRET_KEY
STRIPE_WEBHOOK_SECRET
```

#### Test Cards:

- Success: `4242 4242 4242 4242`
- Decline: `4000 0000 0000 0002`
- Expired: `4000 0000 0000 0069`

---

## File Upload

### UploadThing Configuration

The application uses **UploadThing** for file uploads (primarily event images).

#### Supported File Types:

- Images (JPG, PNG, GIF, WebP)

#### Upload Endpoint:

`/api/uploadthing/`

#### Configuration:

```typescript
// lib/uploadthing.ts
export const OurFileRouter = {
  imageUploader: f({ image: { maxFileSize: "4MB" } })
    .middleware(async ({ req }) => {
      const user = await auth();
      if (!user) throw new Error("Unauthorized");
      return { userId: user.id };
    })
    .onUploadComplete(async ({ metadata, file }) => {
      console.log("File uploaded:", file.url);
    }),
};
```

#### Usage in Components:

```typescript
<UploadButton
  endpoint="imageUploader"
  onClientUploadComplete={(res) => {
    setImageUrl(res[0].url);
  }}
  onUploadError={(error) => {
    console.error(error);
  }}
/>
```

---

## Components Guide

### Shared Components

#### Card Component

Displays event information in card format.

```tsx
<Card event={event} hasOrderLink={boolean} hidePrice={boolean} />
```

#### Collection Component

Displays list of events with pagination.

```tsx
<Collection
  data={events}
  emptyTitle="No Events"
  emptyStateSubtext="Try searching again"
  collectionType="All_Events" | "Events_Organized" | "My_Tickets"
  limit={6}
  page={1}
  totalPages={10}
/>
```

#### EventForm Component

Form for creating/editing events.

```tsx
<EventForm
  userId={string}
  type="Create" | "Update"
  event={event}
  eventId={string}
/>
```

#### FileUploader Component

Handles file uploads.

```tsx
<FileUploader onFieldChange={(url) => setImageUrl(url)} imageUrl={string} />
```

#### Search Component

Search events by title.

```tsx
<Search />
```

#### CategoryFilter Component

Filter events by category.

```tsx
<CategoryFilter />
```

#### Checkout Component

Payment processing component.

```tsx
<Checkout event={event} userId={string} />
```

#### Pagination Component

Navigate through event pages.

```tsx
<Pagination page={number} totalPages={number} urlParamName='page' />
```

---

## Environment Variables

### Required Environment Variables

```bash
# MongoDB Connection
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/database

# Clerk (Authentication)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/

# Stripe (Payments)
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# UploadThing (File Upload)
UPLOADTHING_SECRET=...
UPLOADTHING_APP_ID=...

# Server Configuration
NEXT_PUBLIC_SERVER_URL=http://localhost:3000

# Webhooks
WEBHOOK_SECRET=your_secret_here
```

### Variable Explanations

| Variable                             | Description                     | Example                                          |
| ------------------------------------ | ------------------------------- | ------------------------------------------------ |
| `MONGODB_URI`                        | MongoDB connection string       | `mongodb+srv://user:pass@cluster.mongodb.net/db` |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`  | Public Clerk key                | `pk_test_...`                                    |
| `CLERK_SECRET_KEY`                   | Secret Clerk key (server-only)  | `sk_test_...`                                    |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Public Stripe key               | `pk_test_...`                                    |
| `STRIPE_SECRET_KEY`                  | Secret Stripe key (server-only) | `sk_test_...`                                    |
| `STRIPE_WEBHOOK_SECRET`              | Stripe webhook secret           | `whsec_...`                                      |
| `UPLOADTHING_SECRET`                 | UploadThing secret              | `...`                                            |
| `UPLOADTHING_APP_ID`                 | UploadThing app ID              | `...`                                            |
| `NEXT_PUBLIC_SERVER_URL`             | Application URL                 | `http://localhost:3000`                          |

---

## Running the Application

### Development Mode

```bash
npm run dev
```

Runs the development server with Turbopack enabled.
Application available at: `http://localhost:3000`

### Production Build

```bash
npm run build
npm start
```

### Linting

```bash
npm run lint
```

Runs Next.js linter.

---

## Deployment

### Deploy to Vercel (Recommended)

1. **Push to GitHub:**

```bash
git push origin main
```

2. **Connect to Vercel:**
   - Go to [Vercel Dashboard](https://vercel.com/dashboard)
   - Click "New Project"
   - Import your GitHub repository
   - Select `event-management` project

3. **Configure Environment Variables:**
   - In Vercel dashboard, go to Settings → Environment Variables
   - Add all required environment variables:
     - `MONGODB_URI`
     - `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`
     - `CLERK_SECRET_KEY`
     - `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`
     - `STRIPE_SECRET_KEY`
     - `STRIPE_WEBHOOK_SECRET`
     - `UPLOADTHING_SECRET`
     - `UPLOADTHING_APP_ID`
     - `NEXT_PUBLIC_SERVER_URL` (set to your Vercel domain)
     - `WEBHOOK_SECRET`

4. **Deploy:**
   - Click "Deploy"
   - Wait for deployment to complete
   - Your app is now live!

### Update Webhook URLs

After deployment, update webhook URLs in external services:

**Stripe Webhook:**

- Go to [Stripe Dashboard](https://dashboard.stripe.com/webhooks)
- Update webhook endpoint to: `https://your-domain.com/api/webhook/stripe`
- Select events: `checkout.session.completed`

**Clerk Webhook:**

- Go to Clerk dashboard
- Update webhook endpoint to: `https://your-domain.com/api/webhook/clerk`

### Environment Variables for Production

```bash
NEXT_PUBLIC_SERVER_URL=https://your-domain.com
MONGODB_URI=your_production_mongodb_uri
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_production_clerk_key
CLERK_SECRET_KEY=your_production_clerk_secret
# ... other production secrets
```

---

## Troubleshooting

### Common Errors and Solutions

#### 1. "CastError: Cast to ObjectId failed for value"

**Cause:** Invalid MongoDB ObjectId passed to database query
**Solution:**

- Validate ObjectIds before database operations
- Use `Types.ObjectId.isValid()` to check validity
- Check function already includes validation

#### 2. "Cannot read properties of null (reading '\_id')"

**Cause:** Null data in arrays being mapped
**Solution:**

- Collection component now filters null values
- API functions return empty arrays on error
- Check data before rendering

#### 3. "Stripe webhook not processing orders"

**Cause:** Webhook URL not updated or signing secret wrong
**Solution:**

- Update webhook URL in Stripe dashboard
- Verify `STRIPE_WEBHOOK_SECRET` is correct
- Check webhook is returning status 200

#### 4. "User not found after sign-up"

**Cause:** Clerk webhook didn't trigger
**Solution:**

- Verify Clerk webhook endpoint configured
- Check webhook secret is correct
- Manually trigger webhook from Clerk dashboard

#### 5. "Image upload fails"

**Cause:** UploadThing credentials wrong
**Solution:**

- Verify `UPLOADTHING_APP_ID` and `UPLOADTHING_SECRET`
- Check file size doesn't exceed 4MB
- Check file type is supported (images only)

#### 6. "Connection refused (MongoDB)"

**Cause:** MongoDB connection string invalid or service down
**Solution:**

- Verify `MONGODB_URI` is correct
- Check MongoDB Atlas cluster is running
- Verify IP address is whitelisted
- Check network connectivity

#### 7. "Authentication failing"

**Cause:** Clerk keys incorrect
**Solution:**

- Verify public and secret keys from Clerk dashboard
- Check keys match the correct Clerk instance
- Clear browser cookies and try again
- Check middleware configuration

---

## Contributing

### Code Style Guidelines

1. **TypeScript:** Always use TypeScript for type safety
2. **Components:** Create functional components with hooks
3. **Naming:** Use camelCase for variables/functions, PascalCase for components
4. **Comments:** Add comments for complex logic
5. **Error Handling:** Always handle errors gracefully

### Development Workflow

1. Create a feature branch

```bash
git checkout -b feature/your-feature-name
```

2. Make changes and commit

```bash
git add .
git commit -m "feat: add new feature"
```

3. Push to GitHub

```bash
git push origin feature/your-feature-name
```

4. Create Pull Request
5. Wait for review and merge

### Commit Message Format

```
<type>: <description>

<optional body>
<optional footer>
```

**Types:**

- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation
- `style:` Formatting changes
- `refactor:` Code restructuring
- `perf:` Performance improvements
- `test:` Adding tests

### Best Practices

1. **Keep components small and focused**
2. **Use server actions for database operations**
3. **Validate user input on both client and server**
4. **Use TypeScript for type safety**
5. **Add error handling to all async operations**
6. **Optimize images and bundle size**
7. **Test thoroughly before deployment**
8. **Document new features and APIs**

---

## Additional Resources

### Official Documentation

- [Next.js Documentation](https://nextjs.org/docs)
- [React Documentation](https://react.dev)
- [MongoDB Documentation](https://docs.mongodb.com)
- [Clerk Documentation](https://clerk.com/docs)
- [Stripe Documentation](https://stripe.com/docs)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)

### Tutorials

- [Next.js App Router Guide](https://nextjs.org/docs/app)
- [Clerk Integration Guide](https://clerk.com/docs/quickstarts/nextjs)
- [Stripe Integration](https://stripe.com/docs/stripe-js)

### Community

- [Next.js Discord](https://discord.gg/nextjs)
- [React Community](https://react.dev/community)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/nextjs)

---

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## Support

For issues, questions, or suggestions:

- Open an issue on GitHub
- Check existing issues for solutions
- Review documentation above
- Contact the development team

---

**Last Updated:** May 22, 2026
**Version:** 1.0.0

---

## Quick Reference

### Common Commands

```bash
# Development
npm run dev              # Start dev server

# Production
npm run build            # Build for production
npm start                # Start production server

# Code Quality
npm run lint             # Run linter

# Database
# Connect to MongoDB with MONGODB_URI from .env
```

### Key Endpoints

| Route                 | Method | Description           |
| --------------------- | ------ | --------------------- |
| `/`                   | GET    | Home page with events |
| `/events/create`      | GET    | Create event page     |
| `/events/[id]`        | GET    | Event details         |
| `/events/[id]/update` | GET    | Update event page     |
| `/profile`            | GET    | User profile          |
| `/orders`             | GET    | User orders           |
| `/sign-in`            | GET    | Sign in page          |
| `/sign-up`            | GET    | Sign up page          |

### Database Connections

- **Development:** MongoDB Atlas (Cloud)
- **Production:** MongoDB Atlas (Cloud)

---

For more information, please refer to the respective documentation links or contact the development team.
