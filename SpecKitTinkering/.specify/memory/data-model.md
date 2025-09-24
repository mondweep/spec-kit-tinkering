# Data Model: Google Drive Receipt Parser

## A. Core Entities

### 1. Receipt Entity
```sql
CREATE TABLE receipts (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  drive_file_id TEXT NOT NULL,
  drive_file_url TEXT NOT NULL,
  local_file_name TEXT,
  date TEXT NOT NULL,
  supplier TEXT NOT NULL,
  amount DECIMAL(10,2) NOT NULL,
  purpose TEXT,
  last_four_digits TEXT,
  status TEXT DEFAULT 'processed', -- Values: 'processed', 'error', 'pending', 'review_needed'
  confidence REAL, -- OCR confidence score (0-1)
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

**Fields:**
- `id`: Unique identifier for each receipt
- `drive_file_id`: Google Drive file ID for the receipt image
- `drive_file_url`: URL to access the receipt image in Google Drive
- `local_file_name`: Original filename of the receipt image
- `date`: Purchase date in ISO 8601 format (YYYY-MM-DD)
- `supplier`: Name of the merchant/business
- `amount`: Purchase amount as decimal (10 digits, 2 decimal places)
- `purpose`: Description of the purchase
- `last_four_digits`: Last four digits of the payment card used
- `status`: Processing status of the receipt
- `confidence`: Confidence score from OCR processing (0.0 - 1.0)
- `created_at`: Timestamp when record was created
- `updated_at`: Timestamp when record was last updated

**Indexes:**
```sql
CREATE INDEX idx_receipts_date ON receipts(date);
CREATE INDEX idx_receipts_supplier ON receipts(supplier);
CREATE INDEX idx_receipts_status ON receipts(status);
CREATE INDEX idx_receipts_date_supplier ON receipts(date, supplier);
```

### 2. User Session
```sql
CREATE TABLE user_sessions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  access_token TEXT NOT NULL,
  refresh_token TEXT ENCRYPTED NOT NULL, -- Encrypted for security
  expires_at DATETIME NOT NULL,
  drive_folder_id TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

**Fields:**
- `id`: Unique identifier for session
- `access_token`: OAuth 2.0 access token
- `refresh_token`: OAuth 2.0 refresh token (should be encrypted)
- `expires_at`: Expiration time of access token
- `drive_folder_id`: Selected Google Drive folder to monitor
- `created_at`: Timestamp when session was created
- `updated_at`: Timestamp when session was last updated

### 3. Processing Logs
```sql
CREATE TABLE processing_logs (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  receipt_id INTEGER,
  log_level TEXT NOT NULL, -- 'info', 'warning', 'error'
  message TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (receipt_id) REFERENCES receipts (id) ON DELETE SET NULL
);
```

**Fields:**
- `id`: Unique identifier for log entry
- `receipt_id`: Reference to associated receipt (nullable)
- `log_level`: Severity level of the log entry
- `message`: Log message content
- `created_at`: Timestamp when log was created

## B. Data Relationships

### 1. Receipt to Processing Logs
- One receipt can have multiple processing log entries
- Relationship: receipts.id → processing_logs.receipt_id
- Cardinality: One-to-Many

## C. Data Validation Rules

### 1. Receipt Entity
- `date` must be in valid ISO 8601 format (YYYY-MM-DD)
- `amount` must be a positive decimal
- `supplier` cannot be empty
- `status` must be one of: 'processed', 'error', 'pending', 'review_needed'
- `confidence` must be between 0 and 1 if present

### 2. User Session
- `access_token` and `refresh_token` must be non-empty
- `expires_at` must be a future date when created
- `drive_folder_id` must be a valid Google Drive folder ID format

## D. Sample Data

### 1. Example Receipt Record
```sql
INSERT INTO receipts (
  drive_file_id,
  drive_file_url,
  local_file_name,
  date,
  supplier,
  amount,
  purpose,
  last_four_digits,
  status,
  confidence
) VALUES (
  '1a2b3c4d5e6f7g8h9i0j',
  'https://drive.google.com/file/d/1a2b3c4d5e6f7g8h9i0j/view',
  'receipt_20231015_1234.jpg',
  '2023-10-15',
  'Amazon',
  49.99,
  'Office supplies',
  '1234',
  'processed',
  0.95
);
```

### 2. Example User Session Record
```sql
INSERT INTO user_sessions (
  access_token,
  refresh_token,
  expires_at,
  drive_folder_id
) VALUES (
  'ya29.a0A...',
  '1//0gabc123def456...',
  '2023-10-15T15:30:00Z',
  '0a1b2c3d4e5f6g7h8i9'
);
```

## E. Database Operations

### 1. Common Queries

**Get paginated receipts:**
```sql
SELECT * FROM receipts
ORDER BY date DESC
LIMIT ? OFFSET ?;
```

**Search receipts:**
```sql
SELECT * FROM receipts
WHERE supplier LIKE ? OR purpose LIKE ?
ORDER BY date DESC;
```

**Filter by date range:**
```sql
SELECT * FROM receipts
WHERE date BETWEEN ? AND ?
ORDER BY date DESC;
```

**Get receipt statistics:**
```sql
SELECT 
  COUNT(*) as total_receipts,
  SUM(amount) as total_spent,
  AVG(amount) as average_amount
FROM receipts
WHERE status = 'processed';
```

## F. Migration Scripts

### 1. Initial Schema
```sql
-- Create receipts table
CREATE TABLE receipts (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  drive_file_id TEXT NOT NULL,
  drive_file_url TEXT NOT NULL,
  local_file_name TEXT,
  date TEXT NOT NULL,
  supplier TEXT NOT NULL,
  amount DECIMAL(10,2) NOT NULL,
  purpose TEXT,
  last_four_digits TEXT,
  status TEXT DEFAULT 'processed',
  confidence REAL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Create indexes for receipts
CREATE INDEX idx_receipts_date ON receipts(date);
CREATE INDEX idx_receipts_supplier ON receipts(supplier);
CREATE INDEX idx_receipts_status ON receipts(status);

-- Create user_sessions table
CREATE TABLE user_sessions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  access_token TEXT NOT NULL,
  refresh_token TEXT NOT NULL,
  expires_at DATETIME NOT NULL,
  drive_folder_id TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Create processing_logs table
CREATE TABLE processing_logs (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  receipt_id INTEGER,
  log_level TEXT NOT NULL,
  message TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (receipt_id) REFERENCES receipts (id) ON DELETE SET NULL
);
```

## G. Performance Considerations

### 1. Indexes
- Primary indexes on all frequently queried fields
- Composite indexes for common multi-field queries
- Regular maintenance to optimize index performance

### 2. Data Size
- Expected to handle 10,000+ receipt records efficiently
- Proper pagination to handle large datasets
- Optimization for common query patterns