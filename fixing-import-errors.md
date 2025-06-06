# Fixing Import Errors in Kernel MKII

This document provides solutions to common import errors you might encounter when working with the Kernel MKII project.

## Common Import Errors

### Missing ModelSelector Component

If you see:

```
Module not found: Can't resolve './ModelSelector' 
Module not found: Can't resolve '../models/ModelSelector'
```

The ModelSelector component has been moved to its own file in the admin directory.

#### Solution:

Use the direct import from the correct location:

```typescript
import { ModelSelector } from '@/components/admin/ModelSelector';
```

Or for components within the same directory:

```typescript
import { ModelSelector } from './ModelSelector';
```

### Missing @supabase/auth-helpers-react

If you see:

```
Module not found: Can't resolve '@supabase/auth-helpers-react'
```

This dependency has been removed, and direct usage of the Supabase client is preferred.

#### Solution:

Replace:

```typescript
import { useSupabaseClient } from '@supabase/auth-helpers-react';
```

With:

```typescript
import { supabase } from '@/lib/supabase';
```

And update any calls to `useSupabaseClient()` to use the imported `supabase` client directly.

## Fix for useWorkspace Hook

The useWorkspace hook has been simplified to use the supabase client directly, removing the dependency on @supabase/auth-helpers-react:

```typescript
import { useState, useEffect } from 'react';
import { supabase } from '@/lib/supabase';

// Use the hook like this
const { workspace, loading, error, setWorkspaceId } = useWorkspace();
```

## Fix for AiSettings Component

The AiSettings component now uses an inline ModelSelector or imports it directly:

```typescript
import { ModelSelector } from './ModelSelector';
```

## Fixing Database Issues

If you encounter issues with missing tables or functions, use the fix-database API route:

```typescript
// First check if database needs fixing
const response = await fetch('/api/fix-database');
const data = await response.json();

if (data.needs_fix) {
  // Fix the database
  const fixResponse = await fetch('/api/fix-database', { method: 'POST' });
  const fixData = await fixResponse.json();
  console.log('Fix result:', fixData);
}
```

## Workspace-level OpenAI API Key Storage

The system now supports storing OpenAI API keys at the workspace level with encryption:

1. API keys are stored in the `workspace_settings` table
2. Keys are stored encrypted when the encryption option is selected
3. Only workspace admins and owners can access the API keys
4. localStorage is used as a fallback for backward compatibility 