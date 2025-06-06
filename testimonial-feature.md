# Testimonial Feature Documentation

## Overview

The testimonial feature allows users to collect, manage, and display customer testimonials from marketing assets or create them independently. Testimonials can be linked to source assets (where they were extracted from) or exist as standalone entries.

## Database Schema

### `testimonials` Table

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| id | UUID | Yes | Primary key |
| asset_id | UUID | No | Foreign key to assets table, nullable for standalone testimonials |
| workspace_id | UUID | Yes | Foreign key to workspaces table |
| quote_text | TEXT | Yes | The testimonial text content |
| person_name | TEXT | No | Name of the person giving the testimonial |
| person_title | TEXT | No | Job title of the person |
| company | TEXT | No | Company name |
| is_standalone | BOOLEAN | No | Whether this is a standalone testimonial not tied to an asset |
| published_date | TIMESTAMPTZ | No | When the testimonial was published |
| start_offset | INTEGER | No | Start position in source text (for extracted testimonials) |
| end_offset | INTEGER | No | End position in source text (for extracted testimonials) |
| approved | BOOLEAN | Yes | Whether testimonial is approved for use |
| notes | TEXT | No | Internal notes about the testimonial |
| created_at | TIMESTAMPTZ | Yes | When the testimonial was created |

### `testimonial_asset_links` Table

Many-to-many relationship between testimonials and assets.

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| id | UUID | Yes | Primary key |
| testimonial_id | UUID | Yes | Foreign key to testimonials table |
| asset_id | UUID | Yes | Foreign key to assets table |
| created_at | TIMESTAMPTZ | Yes | When the link was created |
| created_by | UUID | No | User who created the link |

## Access Control

Access to testimonials is controlled through Row-Level Security (RLS) policies:

### Read Access
- Users can view approved testimonials in their workspace
- Workspace admins can view all testimonials (approved and unapproved)
- Global admins can view all testimonials across workspaces

### Write Access
- Workspace admins can create, update, and delete testimonials
- Content creators can create testimonials from assets they have access to
- Global admins have full access to create, update, and delete testimonials

## Testimonial Types

### Asset-Linked Testimonials
- Extracted from marketing assets or linked to existing assets
- Have an `asset_id` referencing the source asset
- Can be linked to multiple assets through the junction table

### Standalone Testimonials
- Created manually without a source asset
- Have `is_standalone = true` and `asset_id = null`
- Associated directly with a workspace via `workspace_id`

## Core Functionality

### Creation

Testimonials can be created in two ways:
1. **Extracted from assets**: AI processing can extract testimonials from uploaded documents
2. **Manual creation**: Users can create standalone testimonials through the UI

### Viewing

The testimonials browse page shows all testimonials with filtering options:
- Filter by approval status
- Filter by asset
- Search by quote text or person name
- Sort by various fields

### Approval Process

1. Unapproved testimonials appear in the approval queue
2. Users with approval permissions can review and approve/reject testimonials
3. Batch approval is supported for multiple testimonials
4. Only approved testimonials are available for use in marketing materials

### Deletion

Testimonials can be deleted with proper authorization:
- Deleting a testimonial also removes all its asset links
- Authentication is refreshed before delete operations to ensure proper permissions

## Common Operations

### Fetching Testimonials

```typescript
// Get all testimonials for a workspace
const { data: testimonials } = await supabase
  .from('testimonials')
  .select('*')
  .eq('workspace_id', workspaceId);

// Get only approved testimonials
const { data: approvedTestimonials } = await supabase
  .from('testimonials')
  .select('*')
  .eq('workspace_id', workspaceId)
  .eq('approved', true);

// Get testimonials for an asset
const { data: assetTestimonials } = await supabase
  .from('testimonials')
  .select('*')
  .eq('asset_id', assetId);

// Get pending testimonials for approval
const { data: pendingTestimonials } = await supabase
  .from('testimonials')
  .select('*')
  .eq('workspace_id', workspaceId)
  .eq('approved', false);
```

### Creating a Testimonial

```typescript
// Create a standalone testimonial
const { data, error } = await supabase
  .from('testimonials')
  .insert({
    quote_text: "This product has transformed our workflow!",
    person_name: "Jane Smith",
    person_title: "CTO",
    company: "Tech Innovators",
    published_date: new Date().toISOString(),
    workspace_id: workspaceId,
    is_standalone: true,
    approved: false,
    notes: "From customer interview"
  });

// Create an asset-linked testimonial
const { data, error } = await supabase
  .from('testimonials')
  .insert({
    quote_text: "Great solution that solved our problems.",
    person_name: "John Doe",
    person_title: "CEO",
    company: "Example Corp",
    asset_id: assetId,
    workspace_id: workspaceId,
    is_standalone: false,
    approved: false
  });
```

### Approving Testimonials

```typescript
// Approve a single testimonial
const { data, error } = await supabase
  .from('testimonials')
  .update({ approved: true })
  .eq('id', testimonialId);

// Batch approve multiple testimonials
const { data, error } = await supabase
  .from('testimonials')
  .update({ approved: true })
  .in('id', testimonialIds);
```

### Deleting Testimonials

```typescript
// First get a refreshed authenticated client
const { data: { session } } = await supabase.auth.getSession();
const token = session.access_token;

// Create authenticated client
const authClient = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY,
  {
    global: {
      headers: {
        Authorization: `Bearer ${token}`,
        apikey: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY
      }
    }
  }
);

// Delete links first
await authClient
  .from('testimonial_asset_links')
  .delete()
  .eq('testimonial_id', testimonialId);

// Then delete the testimonial
await authClient
  .from('testimonials')
  .delete()
  .eq('id', testimonialId);
```

## UI Components

The testimonial feature includes the following UI components:

1. **TestimonialsPage**: Main page for browsing and managing testimonials
2. **ApproveTestimonialsPage**: Page for approving pending testimonials
3. **CreateTestimonialModal**: Modal for creating standalone testimonials
4. **TestimonialDetailView**: Modal for viewing testimonial details
5. **ReviewIntelligenceModal**: Modal for reviewing extracted testimonials from assets

## Regression Testing

Comprehensive regression tests have been implemented to ensure the testimonial feature works correctly:

- Tests for creating both standalone and asset-linked testimonials
- Tests for viewing testimonials with proper filtering
- Tests for the approval workflow
- Tests for deletion with proper authentication
- Tests for linking testimonials to multiple assets

## Recent Updates

### Standalone Testimonials Enhancement (2025-05-17)

- Added `workspace_id` column to directly associate testimonials with workspaces
- Made `asset_id` column nullable to support standalone testimonials
- Added `is_standalone` boolean flag to identify non-asset testimonials
- Updated RLS policies to account for standalone testimonials
- Created an index to improve performance for filtering
- Enhanced UI to show "Standalone" instead of "None - Manually created testimonial"

### Testimonials Enhancement (2025-05-16)

- Removed `product` column from the `testimonials` table
- Created `testimonial_asset_links` table for many-to-many relationships
- Implemented row-level security policies
- Added indexes to improve query performance 