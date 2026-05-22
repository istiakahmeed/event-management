# Setup & Installation Guide

Complete step-by-step guide to set up the Event Management Platform locally or for production deployment.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Local Development Setup](#local-development-setup)
3. [Service Integrations](#service-integrations)
4. [Environment Configuration](#environment-configuration)
5. [Database Setup](#database-setup)
6. [Running the Application](#running-the-application)
7. [Production Deployment](#production-deployment)
8. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### System Requirements

- **Node.js:** 18.0.0 or higher
- **npm:** 9.0.0 or higher (or yarn/pnpm)
- **Git:** For version control
- **macOS/Linux/Windows:** Any OS is supported

### Required Accounts

1. **MongoDB Atlas** - Database hosting
2. **Clerk** - Authentication service
3. **Stripe** - Payment processing
4. **UploadThing** - File upload service

### Verify Installation

```bash
# Check Node.js version
node --version
# Expected: v18.0.0+

# Check npm version
npm --version
# Expected: 9.0.0+

# Check Git version
git --version
```

---

## Local Development Setup

### Step 1: Clone Repository

```bash
git clone https://github.com/istiakahmeed/event-management.git
cd event-management
```

### Step 2: Install Dependencies

```bash
# Using npm
npm install

# Or using yarn
yarn install

# Or using pnpm
pnpm install
```

**Expected Output:**

```
added 500+ packages
audited 600+ packages
```

### Step 3: Create `.env` File

In the project root, create `.env` file:

```bash
# From project root
touch .env
```

Add the following variables (see [Environment Configuration](#environment-configuration) for details):

```bash
# MongoDB
MONGODB_URI=

# Clerk
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/

# Stripe
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=

# UploadThing
UPLOADTHING_SECRET=
UPLOADTHING_APP_ID=

# Server
NEXT_PUBLIC_SERVER_URL=http://localhost:3000

# Webhooks
WEBHOOK_SECRET=your_secret_here
```

### Step 4: Start Development Server

```bash
npm run dev
```

**Expected Output:**

```
▲ Next.js 14.1.0
- Local:        http://localhost:3000
- Environments: .env.local

✓ Ready in 3.2s
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## Service Integrations

### 1. MongoDB Atlas Setup

**Step 1: Create MongoDB Account**

1. Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
2. Click "Try Free"
3. Create account with email/Google
4. Verify email address

**Step 2: Create Cluster**

1. On dashboard, click "Build a Database"
2. Select "M0 Sandbox" (free tier)
3. Choose cloud provider (AWS, GCP, or Azure)
4. Select nearest region
5. Click "Create Cluster"
6. Wait 2-3 minutes for cluster creation

**Step 3: Create Database User**

1. In cluster page, click "Security" → "Database Access"
2. Click "Add Database User"
3. Create username and password (save these!)
4. Click "Add User"

**Step 4: Get Connection String**

1. Click "Database" → "Connect"
2. Choose "Drivers" → "Node.js"
3. Copy connection string
4. Replace `<password>` with your password
5. Replace `<database>` with `evently`

**Example Connection String:**

```
mongodb+srv://username:password@cluster0.abc123.mongodb.net/evently?retryWrites=true&w=majority
```

**Step 5: Update `.env`**

```bash
MONGODB_URI=mongodb+srv://username:password@cluster0.abc123.mongodb.net/evently?retryWrites=true&w=majority
```

**Step 6: Whitelist IP Address**

1. Go to "Network Access"
2. Click "Add IP Address"
3. Click "Allow Access from Anywhere" (for development)
4. In production, add specific IPs

---

### 2. Clerk Setup

**Step 1: Create Clerk Account**

1. Go to [Clerk Dashboard](https://dashboard.clerk.com)
2. Click "Sign up"
3. Create account (email or Google)
4. Verify email

**Step 2: Create Application**

1. Click "Create Application"
2. Choose sign-up method:
   - Email
   - Google OAuth
   - GitHub OAuth (recommended)
3. Click "Create Application"

**Step 3: Get API Keys**

1. Go to "API Keys" section
2. Copy:
   - `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` (public)
   - `CLERK_SECRET_KEY` (secret)

**Step 4: Configure Sign-up/Sign-in**

1. Go to "Customization" → "Sign-in & Sign-up"
2. Configure:
   - Email address required: Yes
   - Username required: Yes
   - Phone number: Optional
3. Click "Save"

**Step 5: Configure Redirects**

1. Go to "Customization" → "Redirects"
2. Set:
   - After sign-in URL: `/`
   - After sign-up URL: `/`

**Step 6: Configure Webhooks** (Important!)

1. Go to "Webhooks" section
2. Click "Add Webhook"
3. Set endpoint URL: `http://localhost:3000/api/webhook/clerk` (for dev)
4. Select events: `user.created`, `user.updated`, `user.deleted`
5. Copy webhook secret
6. Click "Create"

**Step 7: Update `.env`**

```bash
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx
CLERK_SECRET_KEY=sk_test_xxxxx
```

**Step 8: Update Middleware** (Already configured)

```typescript
// middleware.ts - Already includes Clerk
import { authMiddleware } from "@clerk/nextjs";

export default authMiddleware({
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

---

### 3. Stripe Setup

**Step 1: Create Stripe Account**

1. Go to [Stripe Dashboard](https://dashboard.stripe.com)
2. Click "Sign up"
3. Complete profile setup
4. Verify email

**Step 2: Get API Keys**

1. Go to "Developers" → "API Keys"
2. Copy:
   - `Publishable key` (NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY)
   - `Secret key` (STRIPE_SECRET_KEY)

**Step 3: Set Up Webhook**

1. Go to "Developers" → "Webhooks"
2. Click "Add endpoint"
3. Endpoint URL: `http://localhost:3000/api/webhook/stripe` (for dev)
4. Select events:
   - `checkout.session.completed`
   - `charge.failed` (optional)
5. Copy signing secret (STRIPE_WEBHOOK_SECRET)
6. Click "Add endpoint"

**Step 4: Test Mode**

1. Ensure you're in "Test mode" (toggle at top)
2. Use test cards for development:
   - Success: `4242 4242 4242 4242`
   - Decline: `4000 0000 0000 0002`
   - Expiry: Any future date
   - CVC: Any 3 digits

**Step 5: Update `.env`**

```bash
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxxxx
STRIPE_SECRET_KEY=sk_test_xxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxx
```

**Step 6: Testing Webhook Locally**
Use Stripe CLI to test webhooks locally:

```bash
# Install Stripe CLI
# macOS with Homebrew
brew install stripe/stripe-cli/stripe

# Login to Stripe
stripe login

# Listen for webhook events
stripe listen --forward-to localhost:3000/api/webhook/stripe

# Copy webhook signing secret and add to .env
```

---

### 4. UploadThing Setup

**Step 1: Create Account**

1. Go to [UploadThing](https://uploadthing.com)
2. Click "Sign up"
3. Create account (GitHub recommended)
4. Verify email

**Step 2: Create Application**

1. Click "Create your first app"
2. Name your app: "Event Management"
3. Click "Create"

**Step 3: Get API Keys**

1. In dashboard, go to "API Keys"
2. Copy:
   - `App ID` (UPLOADTHING_APP_ID)
   - `Secret` (UPLOADTHING_SECRET)

**Step 4: Configure File Routes**
Already configured in `/lib/uploadthing.ts`:

```typescript
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

**Step 5: Update `.env`**

```bash
UPLOADTHING_APP_ID=xxxxx
UPLOADTHING_SECRET=xxxxx
```

---

## Environment Configuration

### Complete `.env` Template

```bash
# ============================================
# DATABASE
# ============================================
MONGODB_URI=mongodb+srv://username:password@cluster0.abc123.mongodb.net/evently?retryWrites=true&w=majority

# ============================================
# AUTHENTICATION (Clerk)
# ============================================
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx
CLERK_SECRET_KEY=sk_test_xxxxx
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/

# ============================================
# PAYMENT (Stripe)
# ============================================
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxxxx
STRIPE_SECRET_KEY=sk_test_xxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxx

# ============================================
# FILE UPLOAD (UploadThing)
# ============================================
UPLOADTHING_APP_ID=xxxxx
UPLOADTHING_SECRET=xxxxx

# ============================================
# SERVER CONFIGURATION
# ============================================
NEXT_PUBLIC_SERVER_URL=http://localhost:3000

# ============================================
# WEBHOOKS
# ============================================
WEBHOOK_SECRET=your_secret_here
```

### Variable Reference

| Variable                             | Type   | Required | Notes                          |
| ------------------------------------ | ------ | -------- | ------------------------------ |
| `MONGODB_URI`                        | String | ✓        | MongoDB connection string      |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`  | String | ✓        | Public key (exposed to client) |
| `CLERK_SECRET_KEY`                   | String | ✓        | Secret key (server-only)       |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | String | ✓        | Public key (exposed to client) |
| `STRIPE_SECRET_KEY`                  | String | ✓        | Secret key (server-only)       |
| `STRIPE_WEBHOOK_SECRET`              | String | ✓        | Webhook signing secret         |
| `UPLOADTHING_APP_ID`                 | String | ✓        | UploadThing app identifier     |
| `UPLOADTHING_SECRET`                 | String | ✓        | UploadThing signing secret     |
| `NEXT_PUBLIC_SERVER_URL`             | String | ✓        | Application base URL           |
| `WEBHOOK_SECRET`                     | String | ✗        | Optional webhook security      |

**Note:** Variables prefixed with `NEXT_PUBLIC_` are exposed to the client. Never expose secrets here.

---

## Database Setup

### Initial Database Configuration

The application automatically creates collections on first run through Mongoose schemas.

### Collections Created

1. **users** - User accounts
2. **events** - Event listings
3. **categories** - Event categories
4. **orders** - Purchase orders

### Sample Data (Optional)

To seed initial categories:

```typescript
// In your MongoDB shell or through Compass
db.categories.insertMany([
  { name: "Technology" },
  { name: "Business" },
  { name: "Entertainment" },
  { name: "Sports" },
  { name: "Arts & Culture" },
  { name: "Health & Wellness" },
  { name: "Education" },
  { name: "Music" },
]);
```

Or through the app by creating categories via the dashboard.

---

## Running the Application

### Development Server

```bash
npm run dev
```

**Features:**

- Hot reload on code changes
- Turbopack enabled for faster builds
- Development error overlays
- Mock Stripe webhooks (with Stripe CLI)

### Production Build

```bash
# Build for production
npm run build

# Start production server
npm start
```

### Other Commands

```bash
# Run linter
npm run lint

# Run tests (if configured)
npm test

# Clean build cache
rm -rf .next/
npm run build
```

---

## Production Deployment

### Prepare for Production

1. **Update Environment Variables**

   ```bash
   NEXT_PUBLIC_SERVER_URL=https://yourdomain.com
   # Use production API keys from Stripe, Clerk, UploadThing
   ```

2. **Test Production Build**

   ```bash
   npm run build
   npm start
   # Test at http://localhost:3000
   ```

3. **Enable HTTPS**
   - Vercel does this automatically
   - Required for production

### Deploy to Vercel

**Step 1: Push to GitHub**

```bash
git add .
git commit -m "Ready for deployment"
git push origin main
```

**Step 2: Connect to Vercel**

1. Go to [Vercel Dashboard](https://vercel.com)
2. Click "New Project"
3. Import your GitHub repository
4. Select `event-management` project
5. Click "Import"

**Step 3: Set Environment Variables**

1. Go to "Settings" → "Environment Variables"
2. Add all production environment variables
3. Select environment: "Production"

**Step 4: Deploy**

1. Click "Deploy"
2. Wait for build (5-10 minutes)
3. Get your production URL

### Post-Deployment Configuration

**Update Clerk Webhook URL**

1. Go to Clerk dashboard
2. Update webhook endpoint: `https://yourdomain.com/api/webhook/clerk`
3. Save changes

**Update Stripe Webhook URL**

1. Go to Stripe dashboard
2. Go to "Developers" → "Webhooks"
3. Update endpoint: `https://yourdomain.com/api/webhook/stripe`
4. Update signing secret in environment variables

**Update UploadThing Settings**

1. Go to UploadThing dashboard
2. Update allowed origins to include your domain
3. Verify file upload works

---

## Troubleshooting

### Common Setup Issues

#### 1. MongoDB Connection Failed

**Error:** `MongooseConnectionError: connect ECONNREFUSED`

**Solutions:**

- Check MongoDB URI is correct
- Verify IP address is whitelisted in MongoDB Atlas
- Check internet connection
- Wait for cluster to be ready (can take 5 minutes)

```bash
# Test connection
mongosh "mongodb+srv://username:password@cluster.mongodb.net/evently"
```

#### 2. Clerk Keys Not Working

**Error:** `ClerkAuthError: Invalid API key`

**Solutions:**

- Verify correct key (publishable vs secret)
- Check keys are copied completely without spaces
- Regenerate keys if necessary
- Restart development server

#### 3. Stripe Webhook Not Triggering

**Error:** Orders not created after payment

**Solutions:**

- Verify webhook endpoint URL is correct
- Check webhook signing secret matches
- Use Stripe CLI for local testing
- Check webhook logs in Stripe dashboard

```bash
# Test with Stripe CLI
stripe trigger payment_intent.succeeded
```

#### 4. UploadThing Upload Fails

**Error:** `UploadThingError: Unauthorized`

**Solutions:**

- Verify APP_ID and SECRET are correct
- Check file size is under 4MB
- Verify file type is an image
- Regenerate credentials

#### 5. Port 3000 Already in Use

**Error:** `Error: listen EADDRINUSE: address already in use :::3000`

**Solutions:**

```bash
# macOS/Linux - Kill process on port 3000
lsof -i :3000
kill -9 <PID>

# Or use different port
PORT=3001 npm run dev

# Windows - Kill process on port 3000
netstat -ano | findstr :3000
taskkill /PID <PID> /F
```

#### 6. Build Fails with TypeScript Errors

**Error:** `Type 'unknown' is not assignable to type...`

**Solutions:**

```bash
# Clear cache and rebuild
rm -rf .next/
npm run build

# Check TypeScript configuration
npx tsc --noEmit
```

#### 7. Dependencies Installation Issues

**Error:** `npm ERR! code E...`

**Solutions:**

```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and lock file
rm -rf node_modules
rm package-lock.json

# Reinstall dependencies
npm install
```

### Debug Mode

Enable debug logging:

```bash
# Enable debug output
DEBUG=* npm run dev

# Only debug database queries
DEBUG=mongoose:* npm run dev
```

### Getting Help

1. Check error messages carefully
2. Review [Documentation](./DOCUMENTATION.md)
3. Check [API Reference](./API_REFERENCE.md)
4. Search GitHub issues
5. Check service documentation:
   - [Clerk Docs](https://clerk.com/docs)
   - [Stripe Docs](https://stripe.com/docs)
   - [UploadThing Docs](https://docs.uploadthing.com)
   - [MongoDB Docs](https://docs.mongodb.com)

---

## Verification Checklist

After setup, verify everything works:

- [ ] Development server runs without errors
- [ ] Can sign up with Clerk
- [ ] Can log in successfully
- [ ] Can create events
- [ ] Can upload event images
- [ ] Can search and filter events
- [ ] Can view event details
- [ ] Can complete test payment with Stripe test card
- [ ] Order appears in database
- [ ] Profile shows purchased tickets
- [ ] No console errors in browser

---

## Next Steps

1. Review [DOCUMENTATION.md](./DOCUMENTATION.md) for full feature overview
2. Check [API_REFERENCE.md](./API_REFERENCE.md) for API details
3. Start developing features
4. Deploy to production when ready

---

**Document Version:** 1.0.0
**Last Updated:** May 22, 2026
