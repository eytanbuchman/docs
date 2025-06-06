# API Key Management in Kernel MKII

This document explains the improved API key management system in Kernel MKII, which now supports storing OpenAI API keys at the workspace level.

## Key Features

1. **Workspace-Level Storage**: API keys are now persisted in the database at the workspace level
2. **Encryption Support**: Keys can be encrypted using pgcrypto for enhanced security
3. **Backward Compatibility**: Maintains compatibility with localStorage-based key storage
4. **Central Management**: Workspace admins can manage keys for all workspace users
5. **Secure Access Controls**: Only workspace admins can update API keys

## Technical Implementation

### Database Schema

The `workspace_settings` table stores all workspace-level settings, including API keys:

```sql
CREATE TABLE public.workspace_settings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workspace_id UUID NOT NULL,
  key TEXT NOT NULL,
  value TEXT,
  encrypted BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  created_by UUID,
  
  CONSTRAINT workspace_settings_unique_key UNIQUE (workspace_id, key),
  CONSTRAINT fk_workspace_settings_workspace 
  FOREIGN KEY (workspace_id) REFERENCES public.workspaces(id) ON DELETE CASCADE
);
```

### API Endpoints

1. **POST /api/workspace-settings**:
   - Saves a setting to the workspace_settings table
   - Supports encryption of sensitive values
   - Only accessible to workspace admins

2. **GET /api/workspace-settings**:
   - Retrieves settings for a workspace
   - Automatically decrypts encrypted values
   - All workspace users can read settings

3. **GET /api/fix-database**:
   - Checks database structure status
   - Reports on missing tables or functions

4. **POST /api/fix-database**:
   - Creates missing database components
   - Sets up required functions for encryption

### Security Considerations

1. **Encryption**: API keys can be encrypted using pgcrypto's symmetric encryption
2. **Access Controls**: RLS policies ensure only workspace admins can modify keys
3. **Key Rotation**: Support for easy key rotation through the admin interface
4. **Environment Variables**: Server-side encryption key stored in environment variables

## Usage Examples

### Backend Usage

When an API needs to access the OpenAI API, it follows this precedence:

1. Check for key in request headers:

```typescript
const headerApiKey = request.headers.get('x-openai-key');
if (headerApiKey) {
  return headerApiKey;
}
```

2. Look for workspace-level settings:

```typescript
const { data: settings } = await supabase
  .from('workspace_settings')
  .select('value, encrypted')
  .eq('workspace_id', workspaceId)
  .eq('key', 'openai_api_key')
  .single();
  
if (settings?.value) {
  return settings.value;
}
```

### Frontend Component

The `ApiKeysPanel` component provides a UI for managing API keys:

```tsx
<ApiKeysPanel />
```

This component:
- Loads existing API keys from the workspace settings
- Falls back to localStorage for backward compatibility
- Provides encryption toggle for enhanced security
- Saves keys to both workspace settings and localStorage

## Migration Path

For existing users, the system:

1. Checks for localStorage keys on initial load
2. Offers to save them to workspace settings
3. Continues writing to both locations for compatibility

## Troubleshooting

If you encounter database-related issues:

1. Check database status using the ApiKeysPanel in Admin settings
2. Use the RunDatabaseSetupButton to create missing components
3. Manually trigger database fix via `/api/fix-database` endpoint

## For Developers

When implementing new features that require API keys:

1. Use the `getWorkspaceApiKey(workspaceId, keyName)` utility function
2. Always check for request headers first, then workspace settings
3. Use localStorage only as a last resort for backward compatibility
4. Always handle the case where no API key is available

For encrypted values, use the proper API endpoint to retrieve decrypted values rather than directly accessing the database. 