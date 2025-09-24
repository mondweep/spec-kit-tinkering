# Google Drive Receipt Parser - API Specification

## API Overview
The Google Drive Receipt Parser application provides a RESTful API for managing receipt data extracted from Google Drive images. All endpoints follow REST conventions and return JSON responses.

## Authentication
All endpoints except authentication routes require a valid JWT token in the Authorization header:
```
Authorization: Bearer <jwt_token>
```

## Common Response Format
Successful responses follow this format:
```json
{
  "success": true,
  "data": { ... },
  "message": "Optional message"
}
```

Error responses follow this format:
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": "Optional technical details"
  }
}
```

## Endpoints

### Authentication Endpoints

#### POST /api/auth/google
Initiate Google OAuth flow

**Request:**
```json
{
  "redirect_uri": "https://your-app.com/callback"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "auth_url": "https://accounts.google.com/o/oauth2/auth?..."
  }
}
```

#### POST /api/auth/callback
Handle Google OAuth callback

**Request:**
```json
{
  "code": "authorization_code_from_google",
  "state": "state_parameter"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "jwt_token",
    "user": {
      "id": "user_id",
      "email": "user@example.com",
      "name": "User Name"
    }
  }
}
```

#### POST /api/auth/logout
Logout user and invalidate token

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response:**
```json
{
  "success": true,
  "message": "Successfully logged out"
}
```

### Google Drive Integration Endpoints

#### GET /api/drive/folders
Get user's Google Drive folders

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Query Parameters:**
- `parent` (optional): Parent folder ID to list subfolders
- `q` (optional): Query string to filter results

**Response:**
```json
{
  "success": true,
  "data": {
    "folders": [
      {
        "id": "folder_id",
        "name": "Folder Name",
        "mimeType": "application/vnd.google-apps.folder",
        "createdTime": "2023-01-01T00:00:00.000Z",
        "modifiedTime": "2023-01-01T00:00:00.000Z"
      }
    ],
    "nextPageToken": "next_page_token"
  }
}
```

#### POST /api/drive/select-folder
Select a folder for receipt processing

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Request:**
```json
{
  "folderId": "google_drive_folder_id"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "folderId": "google_drive_folder_id",
    "folderName": "Selected Folder Name",
    "status": "selected"
  }
}
```

#### GET /api/drive/status
Get Google Drive integration status

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "connected": true,
    "selectedFolder": {
      "id": "folder_id",
      "name": "Folder Name"
    },
    "lastSync": "2023-01-01T00:00:00.000Z",
    "fileCount": 42,
    "processedCount": 30
  }
}
```

### Receipt Data Endpoints

#### GET /api/receipts
Get paginated list of receipts

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Query Parameters:**
- `page` (default: 1): Page number
- `limit` (default: 20, max: 100): Items per page
- `search` (optional): Search term to match against supplier, purpose, or amount
- `startDate` (optional): Filter by minimum date (YYYY-MM-DD)
- `endDate` (optional): Filter by maximum date (YYYY-MM-DD)
- `supplier` (optional): Filter by specific supplier
- `sortBy` (default: "date"): Sort field (date, supplier, amount, purpose)
- `sortOrder` (default: "desc"): Sort order (asc, desc)

**Response:**
```json
{
  "success": true,
  "data": {
    "receipts": [
      {
        "id": "receipt_id",
        "date": "2023-10-15",
        "supplier": "Amazon",
        "amount": 49.99,
        "currency": "USD",
        "purpose": "Office supplies",
        "fileName": "amazon_receipt_001.jpg",
        "filePath": "/path/to/file/in/drive",
        "lastFourDigits": "1234",
        "status": "processed",
        "confidence": 0.95,
        "createdAt": "2023-10-15T10:30:00.000Z",
        "updatedAt": "2023-10-15T10:30:00.000Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "pages": 8,
      "hasNext": true,
      "hasPrev": false
    }
  }
}
```

#### GET /api/receipts/:id
Get specific receipt details

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "receipt": {
      "id": "receipt_id",
      "date": "2023-10-15",
      "supplier": "Amazon",
      "amount": 49.99,
      "currency": "USD",
      "purpose": "Office supplies",
      "fileName": "amazon_receipt_001.jpg",
      "filePath": "/path/to/file/in/drive",
      "lastFourDigits": "1234",
      "status": "processed",
      "confidence": 0.95,
      "rawText": "Raw OCR text extracted from image",
      "categories": ["Office", "Supplies"],
      "createdAt": "2023-10-15T10:30:00.000Z",
      "updatedAt": "2023-10-15T10:30:00.000Z"
    }
  }
}
```

#### PUT /api/receipts/:id
Update receipt information

**Headers:**
```
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

**Request:**
```json
{
  "date": "2023-10-15",
  "supplier": "Amazon.com",
  "amount": 49.99,
  "purpose": "Office supplies",
  "lastFourDigits": "1234",
  "categories": ["Office", "Supplies"]
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "receipt": {
      "id": "receipt_id",
      "date": "2023-10-15",
      "supplier": "Amazon.com",
      "amount": 49.99,
      "purpose": "Office supplies",
      "lastFourDigits": "1234",
      "status": "processed",
      "updatedAt": "2023-10-15T11:00:00.000Z"
    }
  }
}
```

#### DELETE /api/receipts/:id
Delete a receipt record

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response:**
```json
{
  "success": true,
  "message": "Receipt deleted successfully"
}
```

#### POST /api/receipts/manual
Add a receipt manually (without processing an image)

**Headers:**
```
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

**Request:**
```json
{
  "date": "2023-10-15",
  "supplier": "Local Store",
  "amount": 24.50,
  "currency": "USD",
  "purpose": "Groceries",
  "lastFourDigits": "5678",
  "fileName": "manual_entry",
  "categories": ["Food"]
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "receipt": {
      "id": "new_receipt_id",
      "date": "2023-10-15",
      "supplier": "Local Store",
      "amount": 24.50,
      "currency": "USD",
      "purpose": "Groceries",
      "fileName": "manual_entry",
      "lastFourDigits": "5678",
      "status": "manual",
      "categories": ["Food"],
      "createdAt": "2023-10-15T11:00:00.000Z"
    }
  }
}
```

### Processing Endpoints

#### POST /api/process/trigger
Trigger manual processing of Google Drive receipts

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "jobId": "processing_job_id",
    "status": "started",
    "estimatedTime": 120
  }
}
```

#### GET /api/process/status
Get processing job status

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Query Parameters:**
- `jobId` (optional): Specific job to check, or returns latest job if omitted

**Response:**
```json
{
  "success": true,
  "data": {
    "jobId": "processing_job_id",
    "status": "processing|completed|failed|pending",
    "progress": 0.65,
    "totalFiles": 20,
    "processedFiles": 13,
    "failedFiles": 0,
    "startTime": "2023-10-15T10:00:00.000Z",
    "estimatedEndTime": "2023-10-15T10:02:30.000Z",
    "details": {
      "currentFile": "receipt_007.jpg",
      "message": "Processing file 13 of 20"
    }
  }
}
```

#### GET /api/process/stats
Get processing statistics

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "totalReceipts": 150,
    "processedToday": 5,
    "processingAccuracy": 0.85,
    "totalFilesProcessed": 200,
    "failedProcesses": 3,
    "averageProcessingTime": 8.5
  }
}
```

### Export Endpoints

#### GET /api/export/csv
Download CSV export of receipts

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Query Parameters:**
- `startDate` (optional): Filter by minimum date
- `endDate` (optional): Filter by maximum date
- `supplier` (optional): Filter by specific supplier
- `format` (default: "csv"): Export format (csv, xlsx, json)

**Response:**
```
Content-Type: text/csv or application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename="receipts_YYYYMMDD_HHMMSS.csv"
[CSV or Excel file content]
```

#### POST /api/export/generate
Generate export file (for large datasets)

**Headers:**
```
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

**Request:**
```json
{
  "format": "csv", // csv, xlsx, pdf, json
  "filters": {
    "startDate": "2023-01-01",
    "endDate": "2023-12-31",
    "suppliers": ["Amazon", "Starbucks"],
    "minAmount": 10.00,
    "maxAmount": 100.00
  },
  "columns": ["date", "supplier", "amount", "purpose", "fileName", "lastFourDigits"]
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "exportId": "export_job_id",
    "status": "queued",
    "estimatedTime": 30
  }
}
```

#### GET /api/export/:exportId/status
Check export job status

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "exportId": "export_job_id",
    "status": "completed",
    "downloadUrl": "/api/export/download/export_job_id",
    "fileName": "receipts_YYYYMMDD_HHMMSS.csv",
    "fileSize": 154286,
    "expiresAt": "2023-10-16T10:00:00.000Z"
  }
}
```

#### GET /api/export/download/:exportId
Download generated export file

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response:**
```
Content-Type: appropriate mime type
Content-Disposition: attachment; filename="export_filename.ext"
[File content]
```

### User Preferences Endpoints

#### GET /api/user/preferences
Get user preferences

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "preferences": {
      "autoProcess": true,
      "notificationEmail": "user@example.com",
      "defaultCurrency": "USD",
      "dateFormat": "YYYY-MM-DD",
      "timeZone": "America/New_York",
      "lastSelectedFolder": "folder_id"
    }
  }
}
```

#### PUT /api/user/preferences
Update user preferences

**Headers:**
```
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

**Request:**
```json
{
  "autoProcess": true,
  "notificationEmail": "newemail@example.com",
  "defaultCurrency": "EUR",
  "dateFormat": "DD/MM/YYYY"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "preferences": {
      "autoProcess": true,
      "notificationEmail": "newemail@example.com",
      "defaultCurrency": "EUR",
      "dateFormat": "DD/MM/YYYY",
      "timeZone": "America/New_York",
      "lastSelectedFolder": "folder_id"
    }
  }
}
```

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| AUTH_001 | 401 | Invalid or expired token |
| AUTH_002 | 401 | Insufficient permissions |
| VALIDATION_001 | 400 | Validation error |
| NOT_FOUND_001 | 404 | Resource not found |
| PROCESSING_001 | 422 | Processing error |
| EXPORT_001 | 422 | Export generation error |
| DRIVE_001 | 502 | Google Drive API error |
| INTERNAL_001 | 500 | Internal server error |

## Rate Limiting
- General API endpoints: 100 requests per minute per user
- Processing endpoints: 10 requests per minute per user
- Export endpoints: 5 requests per minute per user

## Webhook Endpoints (Optional)

#### POST /api/webhooks/drive-change
Receive notifications about Google Drive changes (if using push notifications)

**Request:**
```json
{
  "channelId": "channel_id",
  "resourceId": "resource_id",
  "resourceUri": "https://www.googleapis.com/...",
  "token": "token_provided_during_watch",
  "expiration": "timestamp"
}
```

## Security Headers
All API responses include:
- Strict-Transport-Security: max-age=31536000; includeSubDomains
- X-Content-Type-Options: nosniff
- X-Frame-Options: DENY
- X-XSS-Protection: 1; mode=block