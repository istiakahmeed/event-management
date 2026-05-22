# Component Documentation

Complete guide to all components used in the Event Management Platform.

---

## Table of Contents

1. [Shared Components](#shared-components)
2. [UI Components](#ui-components)
3. [Page Components](#page-components)
4. [Component Patterns](#component-patterns)

---

## Shared Components

### Card Component

Displays a single event in card format with image, title, and action buttons.

**Location:** `components/shared/Card.tsx`

**Props:**

```typescript
type CardProps = {
  event: IEvent;
  hasOrderLink?: boolean; // Show "Manage Orders" link (for organizer)
  hidePrice?: boolean; // Hide price display
};
```

**Features:**

- Event image with fallback
- Event title and description
- Organizer name
- Category badge
- Price display or "FREE" badge
- Links to event details
- Action buttons (Buy/Manage Orders)

**Usage:**

```tsx
<Card event={event} hasOrderLink={true} hidePrice={false} />
```

**Styling:** Tailwind CSS, responsive grid layout

---

### Collection Component

Container for displaying a collection of events with pagination.

**Location:** `components/shared/Collection.tsx`

**Props:**

```typescript
type CollectionProps = {
  data: IEvent[];
  emptyTitle: string;
  emptyStateSubtext: string;
  limit: number;
  page: number | string;
  totalPages?: number;
  urlParamName?: string;
  collectionType?: "Events_Organized" | "My_Tickets" | "All_Events";
};
```

**Features:**

- Responsive grid (1 col mobile, 2 col tablet, 3 col desktop)
- Pagination controls
- Empty state display
- Filters null/undefined events
- Handles different collection types

**Usage:**

```tsx
<Collection
  data={events}
  emptyTitle='No Events Found'
  emptyStateSubtext='Try searching again'
  collectionType='All_Events'
  limit={6}
  page={1}
  totalPages={10}
/>
```

---

### Search Component

Search bar for finding events by title.

**Location:** `components/shared/Search.tsx`

**Features:**

- Real-time search input
- Search icon
- Debounced search queries
- Updates URL with search parameter

**Usage:**

```tsx
<Search />
```

**Styling:** Responsive search input with icon

---

### CategoryFilter Component

Dropdown filter for selecting event categories.

**Location:** `components/shared/CategoryFilter.tsx`

**Features:**

- Fetches categories from database
- Dropdown selector
- Updates URL with category parameter
- Shows all categories option

**Usage:**

```tsx
<CategoryFilter />
```

---

### EventForm Component

Form for creating or editing events.

**Location:** `components/shared/EventForm.tsx`

**Props:**

```typescript
type EventFormProps = {
  userId: string;
  type: "Create" | "Update";
  event?: IEvent;
  eventId?: string;
};
```

**Form Fields:**

- Title (required, text)
- Description (required, textarea)
- Location (required, text)
- Event Image (required, file upload)
- Start Date & Time (required, datetime picker)
- End Date & Time (required, datetime picker)
- Category (required, select)
- Price (required, number)
- Is Free (checkbox)
- Event URL (optional, text)

**Features:**

- Form validation with Zod
- Date/time picker with react-datepicker
- Image upload via UploadThing
- Submit button with loading state
- Error display
- Pre-populated data for editing

**Usage (Create):**

```tsx
<EventForm userId={userId} type='Create' />
```

**Usage (Update):**

```tsx
<EventForm userId={userId} type='Update' event={event} eventId={eventId} />
```

---

### FileUploader Component

File upload component for event images using UploadThing.

**Location:** `components/shared/FileUploader.tsx`

**Props:**

```typescript
type FileUploaderProps = {
  onFieldChange: (url: string) => void;
  imageUrl: string;
};
```

**Features:**

- Drag and drop support
- Click to upload
- Image preview
- Replace image option
- Loading state
- Error handling

**Usage:**

```tsx
<FileUploader onFieldChange={(url) => setImageUrl(url)} imageUrl={imageUrl} />
```

---

### Checkout Component

Payment interface for event checkout.

**Location:** `components/shared/Checkout.tsx`

**Props:**

```typescript
type CheckoutProps = {
  event: IEvent;
  userId: string;
};
```

**Features:**

- Event summary display
- Price calculation
- Stripe integration
- "Buy Tickets" button
- Free event registration button
- Error handling

**Usage:**

```tsx
<Checkout event={event} userId={userId} />
```

---

### CheckoutButton Component

Button component that triggers checkout/payment.

**Location:** `components/shared/CheckoutButton.tsx`

**Props:**

```typescript
type CheckoutButtonProps = {
  event: IEvent;
  userId: string;
  hasOrderLink: boolean;
};
```

**Features:**

- Conditional rendering based on hasOrderLink
- "Buy Tickets" or "Manage Orders" text
- Loading state during payment
- Stripe redirect

**Usage:**

```tsx
<CheckoutButton event={event} userId={userId} hasOrderLink={false} />
```

---

### Pagination Component

Navigation component for multi-page results.

**Location:** `components/shared/Pagination.tsx`

**Props:**

```typescript
type PaginationProps = {
  page: number | string;
  totalPages: number;
  urlParamName?: string;
};
```

**Features:**

- Previous/Next buttons
- Page number display
- Query parameter updates
- Disabled state for edge pages
- Responsive design

**Usage:**

```tsx
<Pagination page={1} totalPages={10} urlParamName='page' />
```

---

### Header Component

Navigation header with user menu.

**Location:** `components/shared/Header.tsx`

**Features:**

- Logo/brand name
- Navigation links
- Search bar integration
- User dropdown menu
- Responsive design
- Mobile navigation trigger

**Navigation Links:**

- Home
- All Events
- My Tickets
- Create Event
- Profile
- Organizer Dashboard

**User Menu:**

- Profile link
- Sign out button

**Usage:**

```tsx
<Header />
```

---

### MobileNav Component

Mobile navigation drawer.

**Location:** `components/shared/MobileNav.tsx`

**Features:**

- Hamburger menu button
- Slide-out drawer
- Navigation links
- User profile access
- Close on navigation

**Usage:**

```tsx
<MobileNav />
```

---

### Footer Component

Application footer.

**Location:** `components/shared/Footer.tsx`

**Features:**

- Copyright information
- Quick links
- Social media links
- Contact information

**Usage:**

```tsx
<Footer />
```

---

### NavItems Component

Navigation menu items (reused in header and mobile nav).

**Location:** `components/shared/NavItems.tsx`

**Features:**

- Navigation links
- Active link highlighting
- Authentication status aware

**Usage:**

```tsx
<NavItems />
```

---

### DeleteConfirmation Component

Confirmation dialog for deleting events.

**Location:** `components/shared/DeleteConfirmation.tsx`

**Props:**

```typescript
type DeleteConfirmationProps = {
  eventId: string;
  onDelete: () => void;
};
```

**Features:**

- Alert dialog with confirmation
- Delete button with loading state
- Cancel button
- Error handling

**Usage:**

```tsx
<DeleteConfirmation eventId={eventId} onDelete={handleDelete} />
```

---

### Dropdown Component

Generic dropdown component.

**Location:** `components/shared/Dropdown.tsx`

**Props:**

```typescript
type DropdownProps = {
  value: string;
  onSelect: (value: string) => void;
  items: Array<{ label: string; value: string }>;
  placeholder?: string;
};
```

**Features:**

- Click to open/close
- Keyboard navigation
- Value display
- Search (optional)

**Usage:**

```tsx
<Dropdown
  value={selectedValue}
  onSelect={setSelectedValue}
  items={items}
  placeholder='Select an option'
/>
```

---

## UI Components (Radix UI)

Shadcn/UI components based on Radix UI primitives.

### Button Component

**Location:** `components/ui/button.tsx`

**Variants:**

- `default` - Primary button
- `outline` - Outlined button
- `secondary` - Secondary button
- `ghost` - Text-only button
- `destructive` - Danger button

**Sizes:**

- `default` - Default size
- `sm` - Small
- `lg` - Large
- `icon` - Icon-only

**Usage:**

```tsx
<Button variant='default' size='lg'>
  Click Me
</Button>
```

---

### Input Component

**Location:** `components/ui/input.tsx`

**Features:**

- Text input field
- Placeholder support
- Disabled state
- Error styling

**Usage:**

```tsx
<Input
  type='email'
  placeholder='Enter email'
  value={email}
  onChange={(e) => setEmail(e.target.value)}
/>
```

---

### Select Component

**Location:** `components/ui/select.tsx`

**Features:**

- Dropdown selector
- Multiple options
- Placeholder
- Keyboard navigation

**Usage:**

```tsx
<Select value={value} onValueChange={setValue}>
  <SelectTrigger>
    <SelectValue placeholder='Select option' />
  </SelectTrigger>
  <SelectContent>
    <SelectItem value='option1'>Option 1</SelectItem>
    <SelectItem value='option2'>Option 2</SelectItem>
  </SelectContent>
</Select>
```

---

### Form Component

**Location:** `components/ui/form.tsx`

**Features:**

- React Hook Form integration
- Error display
- Label rendering
- Field composition

**Usage:**

```tsx
<Form {...form}>
  <form onSubmit={form.handleSubmit(onSubmit)}>
    <FormField
      control={form.control}
      name='email'
      render={({ field }) => (
        <FormItem>
          <FormLabel>Email</FormLabel>
          <FormControl>
            <Input {...field} />
          </FormControl>
          <FormMessage />
        </FormItem>
      )}
    />
  </form>
</Form>
```

---

### Sheet Component

**Location:** `components/ui/sheet.tsx`

**Features:**

- Slide-out drawer/panel
- Overlay background
- Close button
- Responsive sizing

**Usage:**

```tsx
<Sheet open={open} onOpenChange={setOpen}>
  <SheetTrigger>Open</SheetTrigger>
  <SheetContent>
    <SheetHeader>
      <SheetTitle>Title</SheetTitle>
    </SheetHeader>
    {/* Content */}
  </SheetContent>
</Sheet>
```

---

### AlertDialog Component

**Location:** `components/ui/alert-dialog.tsx`

**Features:**

- Modal dialog
- Action and cancel buttons
- Overlay background
- Keyboard support

**Usage:**

```tsx
<AlertDialog>
  <AlertDialogTrigger>Delete</AlertDialogTrigger>
  <AlertDialogContent>
    <AlertDialogHeader>
      <AlertDialogTitle>Are you sure?</AlertDialogTitle>
    </AlertDialogHeader>
    <AlertDialogFooter>
      <AlertDialogCancel>Cancel</AlertDialogCancel>
      <AlertDialogAction>Delete</AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>
```

---

### Other UI Components

- **Checkbox** - Checkbox input
- **Label** - Form label
- **Textarea** - Multi-line text input
- **Separator** - Visual divider

---

## Page Components

### Home Page

**Location:** `app/(root)/page.tsx`

**Features:**

- Hero section with call-to-action
- Featured events
- Search and filter
- Pagination

---

### Event Details Page

**Location:** `app/(root)/events/[id]/page.tsx`

**Features:**

- Event header with image
- Event information
- Organizer details
- Checkout component
- Related events
- Share options

---

### Create Event Page

**Location:** `app/(root)/events/create/page.tsx`

**Features:**

- EventForm component
- Authentication check
- Success redirect

---

### Update Event Page

**Location:** `app/(root)/events/[id]/update/page.tsx`

**Features:**

- Pre-populated EventForm
- Authorization check
- Success redirect

---

### Profile Page

**Location:** `app/(root)/profile/page.tsx`

**Features:**

- User information
- My Tickets collection
- Events Organized collection
- Profile editing

---

### Orders Page

**Location:** `app/(root)/orders/page.tsx`

**Features:**

- Order history
- Event details
- Ticket information
- Download/print options (optional)

---

### Sign In Page

**Location:** `app/(auth)/sign-in/[[...sign-in]]/page.tsx`

**Features:**

- Clerk sign-in form
- Social login options
- Sign-up link

---

### Sign Up Page

**Location:** `app/(auth)/sign-up/[[...sign-up]]/page.tsx`

**Features:**

- Clerk sign-up form
- Social signup options
- Terms & conditions
- Sign-in link

---

## Component Patterns

### Server Components

All page components and data-fetching components are Server Components:

```typescript
// app/(root)/page.tsx
export default async function Home({ searchParams }: SearchParamProps) {
  const events = await getAllEvents({
    query: searchParams?.query as string,
    category: searchParams?.category as string,
    page: Number(searchParams?.page) || 1,
    limit: 6,
  });

  return (
    <>
      {/* Server component rendering */}
    </>
  );
}
```

**Benefits:**

- Direct database access
- No API overhead
- Better security
- Automatic caching

---

### Client Components

Interactive components use `"use client"`:

```typescript
// components/shared/Search.tsx
"use client";

import { useState } from "react";

export default function Search() {
  const [query, setQuery] = useState("");
  // Client-side interactivity
  return (/* JSX */);
}
```

---

### Form Components with React Hook Form

```typescript
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";

const form = useForm<EventFormValues>({
  resolver: zodResolver(eventFormSchema),
  defaultValues: {
    title: "",
    description: "",
    // ...
  },
});
```

---

### Error Handling

Components handle errors gracefully:

```typescript
try {
  const result = await createEvent({...});
  // Success handling
} catch (error) {
  setError(error.message);
  // Show error to user
}
```

---

### Loading States

Buttons show loading state during async operations:

```typescript
const [isLoading, setIsLoading] = useState(false);

const handleSubmit = async () => {
  setIsLoading(true);
  try {
    // Submit form
  } finally {
    setIsLoading(false);
  }
};

return (
  <Button disabled={isLoading}>
    {isLoading ? "Loading..." : "Submit"}
  </Button>
);
```

---

### Responsive Design

Components use Tailwind breakpoints:

```tsx
<div className='grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3'>
  {/* Responsive grid */}
</div>
```

**Breakpoints:**

- `sm`: 640px
- `md`: 768px
- `lg`: 1024px
- `xl`: 1280px
- `2xl`: 1536px

---

### Component Composition

Components are composed in a pyramid structure:

```
Page (Server Component)
├── Header
├── Main Content
│   ├── Search
│   ├── Filter
│   └── Collection
│       ├── Card
│       ├── Card
│       └── Card
├── Pagination
└── Footer
```

---

## Component Best Practices

1. **Keep components focused** - Single responsibility principle
2. **Use TypeScript** - Proper typing for props
3. **Handle errors** - Always wrap async operations
4. **Add loading states** - Provide user feedback
5. **Optimize images** - Use Next.js Image component
6. **Memoize when needed** - Use React.memo for expensive renders
7. **Avoid prop drilling** - Use context when needed
8. **Document complex components** - Add JSDoc comments

---

**Document Version:** 1.0.0
**Last Updated:** May 22, 2026
