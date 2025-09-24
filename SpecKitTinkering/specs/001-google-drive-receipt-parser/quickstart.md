# Quickstart Guide: Google Drive Receipt Parser

## Overview
This guide will help you set up and run the Google Drive Receipt Parser application. The application connects to your Google Drive, processes receipt images using OCR, and stores the extracted data in a local SQLite database.

## Prerequisites
- Node.js 18+ installed on your machine
- Google Cloud Project with Google Drive API enabled
- OAuth 2.0 credentials configured for web application

### Setting Up Google Cloud Project
1. Go to the [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the Google Drive API for your project
4. Go to "Credentials" and create OAuth 2.0 credentials
5. Configure the authorized redirect URIs:
   - For development: `http://localhost:5173/auth/callback`
   - For production: `https://yourdomain.com/auth/callback` (adjust as needed)
6. Download the credentials JSON file

## Installation

### 1. Clone the Repository
```bash
git clone <repository-url>
cd google-drive-receipt-parser
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Configure Environment Variables
Create a `.env` file in the project root:
```env
VITE_GOOGLE_CLIENT_ID=your_google_client_id
VITE_GOOGLE_CLIENT_SECRET=your_google_client_secret
VITE_REDIRECT_URI=http://localhost:5173/auth/callback
```

> Note: The `VITE_` prefix is required for Vite to expose these variables to the client-side application.

### 4. Verify Setup
```bash
# Check that all dependencies are properly installed
npm run dev
```

## Development

### Starting the Development Server
```bash
npm run dev
```
This will start the Vite development server at `http://localhost:5173`

### Building for Production
```bash
npm run build
```
This will create optimized production assets in the `dist/` directory.

### Running Tests
```bash
# Run all tests
npm test

# Run unit tests
npm run test:unit

# Run end-to-end tests
npm run test:e2e
```

## Running the Application

1. Start the development server: `npm run dev`
2. Open your browser to `http://localhost:5173`
3. Click the "Connect to Google Drive" button
4. Sign in to your Google account and grant necessary permissions
5. Select the Google Drive folder containing your receipt images
6. The application will automatically process existing images and monitor for new ones
7. Processed receipts will appear in the main interface
8. Export your data using the download functionality

## Key Features

### Authentication
- Secure OAuth 2.0 PKCE flow
- Automatic token refresh
- Encrypted token storage

### Receipt Processing
- Client-side OCR using Tesseract.js
- Batch processing of receipts
- Confidence scoring for extracted data
- Manual correction capability

### Data Management
- Local SQLite database for metadata storage
- Google Drive references for image files
- Full CRUD operations for receipt data

### Export Options
- CSV format export
- Excel format export
- Filtered exports based on date range, supplier, etc.

## Project Structure
```
google-drive-receipt-parser/
├── public/
│   ├── index.html          # Main application entry point
│   └── assets/             # Static assets
├── src/
│   ├── components/         # Reusable UI components
│   ├── services/           # Business logic and API calls
│   ├── utils/              # Utility functions
│   └── models/             # Data models
├── tests/                  # Test files
├── data/                   # SQLite database files
├── dist/                   # Production build output
├── package.json            # Project dependencies and scripts
└── vite.config.js          # Vite configuration
```

## Environment Variables

### Required Variables
- `VITE_GOOGLE_CLIENT_ID`: Your Google OAuth client ID
- `VITE_GOOGLE_CLIENT_SECRET`: Your Google OAuth client secret
- `VITE_REDIRECT_URI`: The redirect URI configured in Google Cloud Console

## Common Issues and Solutions

### OAuth Issues
- Ensure your redirect URI exactly matches what's configured in Google Cloud Console
- Check that your Google account has access to the selected Drive folder
- Verify your Google Drive API is enabled in the Cloud Console

### OCR Processing
- Processing may take 5-15 seconds per receipt depending on file size
- Complex receipt layouts may affect extraction accuracy
- Ensure receipt images are clear and not blurry

### Database Issues
- The SQLite database stores only metadata, not the receipt images themselves
- Images remain in Google Drive, referenced by URL
- The database is stored client-side in the browser

## Customization

### UI Customization
- Modify `public/css/style.css` for global styles
- Update component-specific CSS in the `src/components/` directory

### OCR Configuration
- The Tesseract.js configuration can be modified in `src/services/ocr-service.js`
- Custom training data can be added to improve accuracy for specific receipt types

## Deployment

### Building for Production
```bash
npm run build
```

### Serving the Application
The built application is completely static and can be served by any web server:
- Copy the contents of the `dist/` directory to your web server
- Ensure your server is configured to serve `index.html` for client-side routing

## Support
If you encounter issues:
1. Check the browser's developer console for error messages
2. Verify your Google API credentials and configuration
3. Ensure you have a stable internet connection
4. For persistent issues, consult the documentation or reach out to support