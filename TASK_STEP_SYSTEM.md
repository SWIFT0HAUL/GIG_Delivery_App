# Task Step System Documentation

## Overview
The task step system allows tasks to have multiple steps that users must complete. Each step can be either **informational** (guide users) or **data collection** (collect form data, file uploads, etc.).

## Database Structure

### Table: `task_step_submissions`
Stores user submissions for each task step.

**Columns:**
- `id` (uuid) - Primary key
- `user_id` (uuid) - Reference to auth.users
- `task_id` (uuid) - Reference to custom_tasks
- `step_number` (integer) - The step number
- `submission_data` (jsonb) - Stores form data, file URLs, etc.
- `notes` (text) - Optional user notes
- `submitted_at` (timestamp) - When submitted
- `status` (text) - 'submitted', 'approved', 'rejected'
- `reviewed_by` (uuid) - Admin who reviewed
- `reviewed_at` (timestamp) - When reviewed
- `admin_notes` (text) - Admin review notes

## Task Step Configuration

Steps are configured in the `custom_tasks.redirect_path` JSON field:

### Example: Driver License Upload Task

```json
[
  {
    "step_number": 1,
    "step_name": "Requirements",
    "step_description": "Before uploading, ensure your driver's license is valid, not expired, and clearly visible.",
    "step_type": "informational",
    "path": "https://example.com/license-requirements",
    "is_optional": false
  },
  {
    "step_number": 2,
    "step_name": "Upload License Front",
    "step_description": "Upload a clear photo of the front of your driver's license",
    "step_type": "data_collection",
    "path": "",
    "is_optional": false,
    "fields": [
      {
        "field_name": "license_front",
        "field_type": "file_upload",
        "field_label": "Driver License (Front)",
        "required": true,
        "accept": "image/*,.pdf",
        "bucket_name": "kyc-documents",
        "folder": "driver-licenses"
      }
    ]
  },
  {
    "step_number": 3,
    "step_name": "Upload License Back",
    "step_description": "Upload a clear photo of the back of your driver's license",
    "step_type": "data_collection",
    "path": "",
    "is_optional": false,
    "fields": [
      {
        "field_name": "license_back",
        "field_type": "file_upload",
        "field_label": "Driver License (Back)",
        "required": true,
        "accept": "image/*,.pdf",
        "bucket_name": "kyc-documents",
        "folder": "driver-licenses"
      }
    ]
  }
]
```

## Step Types

### 1. Informational Steps
Used to display instructions, requirements, or guidance.

**Properties:**
- `step_type`: `"informational"`
- `path`: URL or route to display (can be external link)
- User must confirm they've read the information

**Example:**
```json
{
  "step_number": 1,
  "step_name": "Read Instructions",
  "step_description": "Review the requirements carefully",
  "step_type": "informational",
  "path": "https://docs.example.com/requirements",
  "is_optional": false
}
```

### 2. Data Collection Steps
Used to collect information from users via forms.

**Properties:**
- `step_type`: `"data_collection"`
- `fields`: Array of field definitions
- No path required (unless you want to show additional context)

**Field Types:**
- `file_upload` - File upload with Supabase Storage integration
- `text` - Single-line text input
- `textarea` - Multi-line text input
- `number` - Numeric input
- `date` - Date picker

## Field Configuration

### File Upload Field
```json
{
  "field_name": "document_id",
  "field_type": "file_upload",
  "field_label": "ID Document",
  "required": true,
  "accept": "image/*,.pdf",
  "bucket_name": "kyc-documents",
  "folder": "ids"
}
```

### Text Field
```json
{
  "field_name": "full_name",
  "field_type": "text",
  "field_label": "Full Name",
  "required": true,
  "placeholder": "Enter your full legal name"
}
```

### Textarea Field
```json
{
  "field_name": "address",
  "field_type": "textarea",
  "field_label": "Full Address",
  "required": true,
  "placeholder": "Enter your complete address"
}
```

### Number Field
```json
{
  "field_name": "age",
  "field_type": "number",
  "field_label": "Age",
  "required": true,
  "placeholder": "Enter your age"
}
```

### Date Field
```json
{
  "field_name": "date_of_birth",
  "field_type": "date",
  "field_label": "Date of Birth",
  "required": true
}
```

## Usage in Code

### Hook: `useTaskStepSubmission`

```typescript
import { useTaskStepSubmission } from '@/hooks/useTaskStepSubmission';

const { 
  loading,
  submissions,
  submitStep,
  isStepCompleted,
  getStepSubmission,
  fetchSubmissions 
} = useTaskStepSubmission(taskId);

// Check if a step is completed
const completed = isStepCompleted(stepNumber);

// Submit a step
await submitStep({
  step_number: 1,
  submission_data: { license_front: 'url_to_file' },
  notes: 'Optional notes'
});
```

### Component: `TaskStepDialog`

The dialog automatically handles both step types:

```typescript
<TaskStepDialog
  open={stepDialogOpen}
  onOpenChange={setStepDialogOpen}
  step={selectedStep}
  taskId={taskId}
  onStepComplete={handleStepComplete}
/>
```

## Admin Review

Admins can view and review step submissions:

```sql
-- View all submissions for a task
SELECT * FROM task_step_submissions 
WHERE task_id = '<task_id>' 
ORDER BY step_number;

-- Approve a submission
UPDATE task_step_submissions 
SET 
  status = 'approved',
  reviewed_by = '<admin_user_id>',
  reviewed_at = NOW(),
  admin_notes = 'Looks good'
WHERE id = '<submission_id>';

-- Reject a submission
UPDATE task_step_submissions 
SET 
  status = 'rejected',
  reviewed_by = '<admin_user_id>',
  reviewed_at = NOW(),
  admin_notes = 'Document is not clear, please re-upload'
WHERE id = '<submission_id>';
```

## File Storage

Files are uploaded to Supabase Storage with the following structure:

```
{bucket_name}/
  {folder}/
    {user_id}/
      {task_id}/
        {step_number}/
          {field_name}_{timestamp}.{ext}
```

Example:
```
kyc-documents/
  driver-licenses/
    user-123/
      task-456/
        2/
          license_front_1698765432000.jpg
```

## Benefits

1. **Structured Data Collection**: Collect specific information in organized steps
2. **Progress Tracking**: Users can see which steps are completed
3. **File Management**: Automatic file upload to Supabase Storage
4. **Admin Review**: Admins can review and approve/reject submissions
5. **Flexible**: Support both informational and data collection steps
6. **Reusable**: Same system works for any type of task
