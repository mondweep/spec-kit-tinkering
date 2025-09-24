# API Contracts: Google Drive Receipt Parser

## Authentication API

### Initiate Google OAuth
```
GET /auth/google
```
Initiates the Google OAuth flow.

**Request:**
- No request body required
- Redirects to Google consent screen

**Response:**
- Redirect to Google OAuth consent screen

---

### OAuth Callback Handler
```
GET /auth/callback
```
Handles the OAuth callback and completes authentication.

**Request Query Parameters:**
- `code` (string): Authorization code from Google
- `state` (string): State parameter for CSRF protection

**Response:**
- Sets authentication tokens in browser storage
- Redirects to main application

---

### Logout
```
POST /auth/logout
```
Logs out the user and clears authentication tokens.

**Request Headers:**
```
Content-Type: application/json
```

**Response:**
```json
{
  "success": true,
  "message": "Successfully logged out"
}
```

## Google Drive API Integration

### Get User's Drive Folders
```
GET /api/drive/folders
```
Retrieves a list of the user's Google Drive folders.

**Request Headers:**
```
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `parentId` (optional, string): Parent folder ID to list subfolders
- `pageToken` (optional, string): Token for pagination

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

---

### Select Drive Folder
```
POST /api/drive/select-folder
```
Selects a Google Drive folder for receipt processing.

**Request Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
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
    "folderName": "Selected Folder Name"
  }
}
```

---

### Get Drive Integration Status
```
GET /api/drive/status
```
Retrieves the status of Google Drive integration.

**Request Headers:**
```
Authorization: Bearer <access_token>
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

## Receipt Management API

### Get Receipts
```
GET /api/receipts
```
Retrieves paginated list of receipts from local database.

**Query Parameters:**
- `page` (optional, integer): Page number (default: 1)
- `limit` (optional, integer): Items per page (default: 20, max: 100)
- `search` (optional, string): Search term to match against supplier or purpose
- `startDate` (optional, string): Filter by minimum date (YYYY-MM-DD)
- `endDate` (optional, string): Filter by maximum date (YYYY-MM-DD)
- `supplier` (optional, string): Filter by specific supplier
- `status` (optional, string): Filter by processing status

**Response:**
```json
{
  "success": true,
  "data": {
    "receipts": [
      {
        "id": 1,
        "drive_file_id": "file_id_from_google_drive",
        "drive_file_url": "https://drive.google.com/file/d/file_id/view",
        "local_file_name": "receipt_20231015.jpg",
        "date": "2023-10-15",
        "supplier": "Amazon",
        "amount": 49.99,
        "purpose": "Office supplies",
        "last_four_digits": "1234",
        "status": "processed",
        "confidence": 0.95,
        "created_at": "2023-10-15T10:30:00.000Z",
        "updated_at": "2023-10-15T10:30:00.000Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "pages": 8
    }
  }
}
```

---

### Get Single Receipt
```
GET /api/receipts/:id
```
Retrieves details of a specific receipt.

**Path Parameter:**
- `id` (integer): Receipt ID

**Response:**
```json
{
  "success": true,
  "data": {
    "receipt": {
      "id": 1,
      "drive_file_id": "file_id_from_google_drive",
      "drive_file_url": "https://drive.google.com/file/d/file_id/view",
      "local_file_name": "receipt_20231015.jpg",
      "date": "2023-10-15",
      "supplier": "Amazon",
      "amount": 49.99,
      "purpose": "Office supplies",
      "last_four_digits": "1234",
      "status": "processed",
      "confidence": 0.95,
      "created_at": "2023-10-15T10:30:00.000Z",
      "updated_at": "2023-10-15T10:30:00.000Z"
    }
  }
}
```

---

### Update Receipt
```
PUT /api/receipts/:id
```
Updates details of a specific receipt.

**Path Parameter:**
- `id` (integer): Receipt ID

**Request Headers:**
```
Content-Type: application/json
```

**Request Body:**
```json
{
  "date": "2023-10-15",
  "supplier": "Amazon.com",
  "amount": 49.99,
  "purpose": "Office supplies",
  "last_four_digits": "1234"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "receipt": {
      "id": 1,
      "drive_file_id": "file_id_from_google_drive",
      "drive_file_url": "https://drive.google.com/file/d/file_id/view",
      "local_file_name": "receipt_20231015.jpg",
      "date": "2023-10-15",
      "supplier": "Amazon.com",
      "amount": 49.99,
      "purpose": "Office supplies",
      "last_four_digits": "1234",
      "status": "processed",
      "confidence": 0.95,
      "created_at": "2023-10-15T10:30:00.000Z",
      "updated_at": "2023-10-15T11:00:00.000Z"
    }
  }
}
```

---

### Delete Receipt
```
DELETE /api/receipts/:id
```
Deletes a specific receipt.

**Path Parameter:**
- `id` (integer): Receipt ID

**Response:**
```json
{
  "success": true,
  "message": "Receipt deleted successfully"
}
```

---

### Process Drive Receipts
```
POST /api/receipts/process
```
Triggers batch processing of new receipt images from Google Drive.

**Request Headers:**
```
Content-Type: application/json
```

**Request Body:**
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
    "jobId": "processing_job_id",
    "status": "started",
    "estimatedTime": 120,
    "totalFiles": 5
  }
}
```

---

### Get Processing Status
```
GET /api/receipts/process/:jobId/status
```
Gets the status of a receipt processing job.

**Path Parameter:**
- `jobId` (string): Processing job ID

**Response:**
```json
{
  "success": true,
  "data": {
    "jobId": "processing_job_id",
    "status": "processing|completed|failed|pending",
    "progress": 0.6,
    "totalFiles": 5,
    "processedFiles": 3,
    "failedFiles": 0,
    "startTime": "2023-10-15T10:00:00.000Z",
    "estimatedEndTime": "2023-10-15T10:02:00.000Z"
  }
}
```

## Export API

### Export Receipts (CSV)
```
GET /api/export/csv
```
Downloads a CSV file of receipt data.

**Query Parameters:**
- `startDate` (optional, string): Filter by minimum date (YYYY-MM-DD)
- `endDate` (optional, string): Filter by maximum date (YYYY-MM-DD)
- `supplier` (optional, string): Filter by specific supplier
- `status` (optional, string): Filter by processing status

**Response:**
- Content-Type: text/csv
- Content-Disposition: attachment; filename="receipts_YYYYMMDD_HHMMSS.csv"

---

### Export Receipts (Excel)
```
GET /api/export/xlsx
```
Downloads an Excel file of receipt data.

**Query Parameters:**
- `startDate` (optional, string): Filter by minimum date (YYYY-MM-DD)
- `endDate` (optional, string): Filter by maximum date (YYYY-MM-DD)
- `supplier` (optional, string): Filter by specific supplier
- `status` (optional, string): Filter by processing status

**Response:**
- Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
- Content-Disposition: attachment; filename="receipts_YYYYMMDD_HHMMSS.xlsx"

---

### Export Receipts (JSON)
```
GET /api/export/json
```
Downloads a JSON file of receipt data.

**Query Parameters:**
- `startDate` (optional, string): Filter by minimum date (YYYY-MM-DD)
- `endDate` (optional, string): Filter by maximum date (YYYY-MM-DD)
- `supplier` (optional, string): Filter by specific supplier
- `status` (optional, string): Filter by processing status

**Response:**
- Content-Type: application/json
- Content-Disposition: attachment; filename="receipts_YYYYMMDD_HHMMSS.json"
```

## Error Response Format

All error responses follow this format:
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

## Common Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| AUTH_001 | 401 | Invalid or expired token |
| AUTH_002 | 403 | Insufficient permissions |
| VALIDATION_001 | 400 | Validation error in request |
| NOT_FOUND_001 | 404 | Resource not found |
| PROCESSING_001 | 422 | Processing error |
| DATABASE_001 | 500 | Database error |
| DRIVE_001 | 502 | Google Drive API error |
| INTERNAL_001 | 500 | Internal server error |

## Rate Limiting

- API requests are limited to 100 requests per minute per user
- Exceeding rate limits results in a 429 status code