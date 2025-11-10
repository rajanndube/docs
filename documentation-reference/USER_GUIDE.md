# StringBoot Dashboard User Guide

Complete guide to all available actions on the StringBoot Dashboard UI.

---

## Table of Contents

1. [Authentication](#authentication)
2. [Dashboard Overview](#dashboard-overview)
3. [Applications Management](#applications-management)
4. [String Resources](#string-resources)
5. [Languages](#languages)
6. [API Tokens](#api-tokens)
7. [FAQs](#faqs)
8. [Settings](#settings)

---

## Authentication

### Register a New Account

1. Navigate to `/register`
2. Enter your email address
3. Create a strong password (minimum 8 characters)
4. Click **Register**
5. Optionally, use **Google Sign In** button for OAuth authentication

### Login to Your Account

1. Navigate to `/login`
2. Enter your email and password
3. Click **Sign In**
4. Alternatively, click **Sign in with Google** for OAuth

### Reset Password

1. On the login page, click **Forgot Password?**
2. Enter your email address
3. Follow the instructions sent to your email
4. Create a new password

### Link Google Account

1. Go to **Settings** (accessible from sidebar)
2. Find the **Account Linking** section
3. Click **Link Google Account**
4. Follow the OAuth flow to connect your account

**Note:** Email verification is required for certain features.

---

## Dashboard Overview

### Access Dashboard

- Navigate to `/dashboard` or `/`
- View key statistics:
  - Total Applications count
  - Total String Resources count
  - Active Languages count
  - Active API Tokens count

### Quick Actions from Dashboard

Click any of these shortcuts:
- **Create New App** → Navigate to new app creation
- **Add String Resource** → Navigate to string creation
- **Add New Language** → Navigate to language creation
- **Generate API Token** → Navigate to token creation

### View Recent Activity

The dashboard displays recent actions:
- String additions
- Language additions
- API token creation
- App creation

---

## Applications Management

### Create a New Application

1. Navigate to **Applications** (`/apps`)
2. Click **New App** button
3. Fill in the form:
   - **App Name** (required)
   - **Description** (optional)
4. Click **Create Application**

### View Application Details

1. Go to **Applications** page
2. Click on any application card
3. View tabs:
   - **Overview** - App details, statistics
   - **Languages** - All configured languages
   - **Strings** - String resources
   - **API Tokens** - Associated tokens

### Edit Application Settings

1. Open an application
2. Click **Settings** button
3. Modify:
   - App name
   - Description
4. Click **Save Changes**

### Delete an Application

1. Navigate to **Applications**
2. Find the application
3. Click the delete icon
4. Confirm deletion in the dialog

**Warning:** Deleting an app removes all associated strings, languages, and tokens.

---

## String Resources

### Create a Single String

1. Navigate to **Strings** (`/strings`)
2. Select an application from the dropdown
3. Click **New String** button
4. Fill in the form:
   - **String Key** (required, max 100 characters)
   - **Language** (select from dropdown)
   - **String Value** (required)
   - **Description** (optional)
5. Click **Create String Resource**

### Bulk Upload Strings

#### Prerequisites
- Have an application created
- Prepare a CSV, XML, or JSON file with string data

#### Steps
1. Navigate to **Strings** page
2. Select your application
3. Click **Bulk Upload** button
4. Choose your file (CSV, XML, or JSON)
5. File format requirements:

   **CSV Format:**
   ```csv
   key,value,languageCode,description
   welcome_message,Welcome!,en,Greeting message
   goodbye_message,Goodbye!,en,Farewell message
   ```

   **JSON Format:**
   ```json
   [
     {
       "key": "welcome_message",
       "value": "Welcome!",
       "languageCode": "en",
       "description": "Greeting message"
     }
   ]
   ```

6. Review the data preview
7. Handle any validation errors or duplicates
8. Click **Upload** to complete

#### Bulk Upload Features
- **S3 Integration** - Cloud-native file processing
- **Real-time Progress** - WebSocket-powered progress tracking
- **Duplicate Resolution** - Bulk resolution system for conflicts
- **Auto-format Conversion** - Automatic CSV/XML/JSON conversion
- **Resume Capability** - Recover from interruptions
- **Error Resolution** - Fix validation errors before upload

### View and Filter Strings

1. Go to **Strings** page
2. Use filters:
   - **Application** - Select specific app
   - **Language** - Filter by language code
   - **Search** - Search by key or value
3. Switch between tabs:
   - **All Strings** - Complete list
   - **Recently Modified** - Last 7 days

### Edit a String

1. Navigate to **Strings**
2. Find the string in the table
3. Click the **Edit** icon
4. Modify:
   - String value
   - Description
5. Click **Save Changes**

### Delete Strings

**Single Delete:**
1. Find the string in the table
2. Click the delete icon
3. Confirm deletion

**Bulk Delete:**
1. Select multiple strings using checkboxes
2. Click **Delete Selected** button
3. Confirm bulk deletion

### Export Strings

1. Navigate to **Strings** page
2. Apply desired filters
3. Click **Export** button
4. Choose format (CSV or JSON)
5. Download begins automatically

---

## Languages

### Add a New Language (Basic)

1. Navigate to **Languages** (`/languages`)
2. Select an application
3. Click **Add Language**
4. Fill in the form:
   - **Language Code** - Select from standard codes (e.g., en, es, fr, hi)
   - **Language Name** - Auto-filled based on code selection
   - **Make Default** - Check if this should be the default language
5. Click **Add Language**

**Note:** If marked as default, sample strings will be automatically created.

### Add Language with AI Translation

#### Prerequisites
- OpenAI API key configured in Settings
- At least one existing language with strings

#### Steps
1. Navigate to **Languages**
2. Click **Add Language**
3. Fill in language details
4. Check **Pre-fill with AI-translated strings from existing language**
5. Select **Source Language** from dropdown
6. Click **Add Language**
7. Review AI-generated translations:
   - Each translation shows source and target
   - Approve/reject individual translations
   - Edit translations before saving
   - View translation progress in real-time
8. Options during review:
   - **Approve All** - Accept all translations
   - **Download CSV** - Export translations
   - **Edit** - Modify individual translations
9. Click **Save Approved Translations**

**Features:**
- Real-time progress tracking via WebSocket
- Inline translation with progress indicator
- Transaction journal for tracking operations
- Submission report with success/failure details

### Set Default Language

1. Go to **Languages** page
2. Select an application
3. Find the language in the table
4. Click **Set Default** button
5. Confirmation toast appears

**Note:** Only one language can be default per application.

### Activate/Deactivate Language

1. Navigate to **Languages**
2. Find the language in the table
3. Toggle the **Active/Inactive** switch
4. Status updates immediately

### Edit Language

1. Go to **Languages** page
2. Click the **Edit** icon for a language
3. Modify:
   - Language name
   - Active status
   - Default status
4. Click **Save Changes**

### Delete Language

1. Navigate to **Languages**
2. Click the delete icon for a language
3. Confirm deletion in the dialog

**Warning:**
- This permanently deletes all strings for this language
- Cannot delete the default language
- Two-step process: strings deleted first, then language

---

## API Tokens

### Create API Token

1. Navigate to **API Tokens** (`/tokens`)
2. Select an application
3. Click **New Token**
4. Fill in the form:
   - **Token Name** (required, max 50 characters)
   - **Token Type**:
     - **Standard** - Server-to-server API access
     - **Plugin** - Browser extensions/plugins (auto-bypass CORS)
   - **Expiry Date** (optional) - Set expiration date
5. Click **Create API Token**
6. **Important:** Copy the token immediately from the dialog
7. Click **Done**

**Security Note:** The token is only shown once. Store it securely.

### View Tokens

1. Navigate to **API Tokens**
2. Select **All Applications** or specific app
3. View token details:
   - Name
   - Token (masked)
   - Application
   - Type (Standard/Plugin)
   - Status (Active/Inactive)
   - Created date
   - Expiry date

### Copy Token

1. Find the token in the table
2. Click the **Copy** icon
3. Token copied to clipboard

### Revoke Token

1. Navigate to **API Tokens**
2. Find the token
3. Click the **Revoke** icon (circular arrow)
4. Confirm revocation

**Note:** Revoked tokens cannot be reactivated.

### Delete Token

1. Navigate to **API Tokens**
2. Find the token
3. Click the **Delete** icon (trash)
4. Confirm deletion

### Revoke All Tokens (for an app)

1. Select a specific application
2. Click **Revoke All** button
3. Confirm action

**Warning:** This revokes all tokens for the selected application.

---

## FAQs

### Create a New FAQ

1. Navigate to **FAQs** (`/faqs`)
2. Select an application
3. Click **New FAQ** button
4. Fill in the form:
   - **Question** (required)
   - **Answer** (required)
   - **Language Code** (required)
   - **Tags** (optional, comma-separated)
5. Click **Save FAQ**

### Bulk Import FAQs

#### Prerequisites
- Have an application created
- Prepare CSV or JSON file

#### Steps
1. Navigate to **FAQs**
2. Select application
3. Click **Bulk Import** button
4. Upload file (CSV or JSON)
5. File format:

   **CSV:**
   ```csv
   question,answer,languageCode,tags
   "What is StringBoot?","A string management platform",en,"general,about"
   ```

   **JSON:**
   ```json
   [
     {
       "question": "What is StringBoot?",
       "answer": "A string management platform",
       "languageCode": "en",
       "tags": ["general", "about"]
     }
   ]
   ```

6. Review preview and handle duplicates
7. Click **Import**

### View and Filter FAQs

1. Go to **FAQs** page
2. Use filters:
   - **Application** - Select app
   - **Language** - Filter by language
   - **Tags** - Filter by specific tag
   - **Search** - Search in questions/answers
3. Switch tabs:
   - **All FAQs**
   - **Recently Modified** (last 7 days)

### Edit FAQ

1. Find the FAQ in the list
2. Click the **Edit** icon
3. Modify question, answer, or tags
4. Click **Save Changes**

### Delete FAQs

**Single Delete:**
1. Find FAQ in the list
2. Click delete icon
3. Confirm deletion

**Bulk Delete:**
1. Select multiple FAQs using checkboxes
2. Click **Delete Selected**
3. Confirm bulk deletion

### Export FAQs

1. Navigate to **FAQs**
2. Apply filters if needed
3. Click **Export** button
4. Choose format (CSV or JSON)
5. Download begins automatically

---

## Settings

### Configure OpenAI API Key

#### Why Configure?
Required for AI translation features when adding new languages.

#### Steps
1. Navigate to **Settings** (`/settings`)
2. Find **API Settings** section
3. Enter your OpenAI API key:
   - Format: Starts with `sk-`, `sk-live_`, `sk-service_`, or `sk-svcacct-`
   - Get one from [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
4. Click eye icon to show/hide key
5. Click **Save Settings**

**Security Note:**
- API key is stored locally in your browser
- Never sent to StringBoot servers
- Only used for translation requests

### Manage Account Linking

1. Go to **Settings**
2. Find **Account Linking** section
3. Link or unlink your Google account
4. View linked provider status

---

## Additional Features

### Offline Mode

The dashboard supports offline functionality:
- Yellow banner appears when backend is unavailable
- Changes saved locally
- Syncs when connection restored

### Real-time Updates

- WebSocket integration for live progress
- Notification system for background tasks
- Auto-refresh for bulk operations

### Search and Filtering

Most pages support:
- Text search
- Multi-level filtering
- Sort by various fields
- Recently modified views

### Keyboard Shortcuts

- `Ctrl/Cmd + K` - Quick search (if enabled)
- `Esc` - Close dialogs/modals

---

## Tips and Best Practices

### Application Management
- Use descriptive names for apps
- Add detailed descriptions for team clarity
- Delete unused apps to keep workspace clean

### String Resources
- Use consistent key naming conventions (e.g., `screen.action.element`)
- Add descriptions for complex strings
- Use bulk upload for large datasets
- Export regularly for backups

### Languages
- Set up default language first
- Use AI translation as starting point, then review
- Keep language codes consistent (use ISO 639-1)
- Don't delete languages with active string usage

### API Tokens
- Use descriptive names indicating purpose/environment
- Set expiry dates for security
- Use Plugin type only for browser-based integrations
- Revoke unused tokens immediately
- Store tokens securely (use environment variables)

### FAQs
- Tag FAQs for easy categorization
- Keep questions concise and clear
- Use bulk import for initial setup
- Export for documentation purposes

### Security
- Enable email verification
- Link Google account for 2FA-like protection
- Regularly rotate API tokens
- Use strong passwords (8+ characters)

---

## Troubleshooting

### Cannot Create String
- Ensure application is selected
- Verify language exists in the app
- Check string key doesn't already exist
- Validate all required fields

### Bulk Upload Failing
- Verify file format matches template
- Check for duplicate keys
- Ensure language codes exist
- Review validation errors in preview

### AI Translation Not Available
- Verify OpenAI API key is configured in Settings
- Check API key format is correct
- Ensure you have at least one existing language
- Verify internet connection

### Token Not Working
- Check if token is active (not revoked)
- Verify expiry date hasn't passed
- Ensure using correct app ID in requests
- Check token type matches use case

### Language Won't Delete
- Cannot delete default language (set another as default first)
- Ensure all strings for language are deleted
- Check if language is used in active configurations

---

## Support and Documentation

For technical documentation and API reference, see:
- **[API_REFERENCE.md](docs/API_REFERENCE.md)** - Complete API documentation
- **[COMPONENTS.md](docs/COMPONENTS.md)** - UI component library
- **[ARCHITECTURE.md](docs/ARCHITECTURE.md)** - Technical architecture
- **[DESIGN_SYSTEM.md](docs/DESIGN_SYSTEM.md)** - Design guidelines

For issues or feature requests:
- Create an issue on the GitHub repository
- Contact your system administrator
- Refer to backend logs for API errors

---

## Keyboard Navigation

- Use `Tab` to navigate between form fields
- `Enter` to submit forms
- `Esc` to close modals
- Arrow keys for table navigation

---

**Last Updated:** Based on current codebase analysis
**Version:** React 18 + TypeScript + Vite 5
