# Setup & Configuration Guide

Complete step-by-step guide to configure the AI UGC Content Engine workflow in your n8n instance.

## Prerequisites

Before starting, ensure you have:
- n8n instance (self-hosted or n8n Cloud)
- Google Workspace (Gmail, Sheets, Drive)
- Arcads API account and credentials
- OpenAI API key
- Slack workspace
- Browser with access to n8n UI

## Step 1: Create Google Sheets

### Intake Sheet
1. Create a new Google Sheet named "Intake"
2. Add columns (headers in row 1):
   - `Brand Name`
   - `Product Name`
   - `Tagline`
   - `Image URL`
   - `Status`

3. Add sample data:
```
Brand Name    | Product Name      | Tagline                    | Image URL                  | Status
Nike          | Air Max 90        | Classic comfort redefined  | https://example.com/1.jpg  | READY
Apple         | iPhone 15 Pro     | Pro power in your hand     | https://example.com/2.jpg  | READY
```

### Output Sheet
1. Create a new Google Sheet named "AI Content Engine"
2. n8n will create columns automatically, but you can pre-create:
   - All Intake columns
   - `Video URL 9:16`, `Video URL 1:1`, `Video URL 16:9`
   - `TikTok Caption`, `TikTok Hashtags`
   - `Instagram Caption`, `Instagram Hashtags`
   - `YouTube Caption`, `YouTube Hashtags`
   - `Status`

### Note the Sheet IDs
1. Copy the Intake sheet ID from the URL: `https://docs.google.com/spreadsheets/d/{SHEET_ID}/...`
2. Copy the Output sheet ID similarly
3. Save these values for later

## Step 2: Create Google Drive Folder

1. In Google Drive, create a new folder named "AI AUTOMATION"
2. Right-click → Share → Change to "Restricted" if needed
3. Copy the folder ID from the URL: `https://drive.google.com/drive/folders/{FOLDER_ID}`
4. Save this value for later

## Step 3: Configure n8n Credentials

### 3a. Google Sheets OAuth2

1. In n8n, go to **Credentials** → **New**
2. Search for "Google Sheets" → Select **Google Sheets OAuth2 API**
3. Click "Sign in with Google"
4. Select your Google account
5. Grant permissions for:
   - View and manage spreadsheets
   - View and manage files in Google Drive
6. Save credential as "Google Sheets OAuth2 API"

### 3b. Google Drive OAuth2

1. In n8n, go to **Credentials** → **New**
2. Search for "Google Drive" → Select **Google Drive OAuth2 API**
3. Click "Sign in with Google"
4. Use the same Google account
5. Grant permissions for file access
6. Save credential as "Google Drive OAuth2 API"

### 3c. Arcads Basic Auth

1. In n8n, go to **Credentials** → **New**
2. Search for "Basic Auth" → Select **HTTP Basic Auth**
3. Enter:
   - **Username**: Your Arcads API username
   - **Password**: Your Arcads API key/password
4. Save credential as "Arcads API Basic Auth"

### 3d. OpenAI API

1. In n8n, go to **Credentials** → **New**
2. Search for "OpenAI" → Select **OpenAI API**
3. Enter:
   - **API Key**: Your OpenAI API key from https://platform.openai.com/api-keys
4. Save credential as "OpenAI API"

**Note**: Ensure account has available credits

### 3e. Slack OAuth2

1. In n8n, go to **Credentials** → **New**
2. Search for "Slack" → Select **Slack OAuth2 API**
3. Click "Sign in with Slack"
4. Select your Slack workspace
5. Grant permissions for:
   - Send messages to channels
   - Read channel information
6. Save credential as "Slack OAuth2 API"

## Step 4: Import Workflow

1. In n8n, go to **Workflows** → **New** → **Import from file**
2. Select `workflow.json` from this repository
3. The workflow will be imported with placeholder credentials

## Step 5: Replace Placeholders

The workflow contains placeholders that need to be replaced with your actual values:

### 5a. Google Sheet IDs

1. Open the workflow editor
2. Click each node with a spreadsheet reference and replace:
   - `{{ GOOGLE_SHEET_ID_INTAKE }}` → Your actual Intake sheet ID
   - `{{ GOOGLE_SHEET_ID_OUTPUT }}` → Your actual Output sheet ID

**Nodes to update**:
- Google Sheets Trigger
- Read All Rows
- Write Output Sheet
- Mark Intake DONE

### 5b. Google Drive Folder ID

1. Click nodes that reference Google Drive:
   - Replace `{{ GOOGLE_DRIVE_FOLDER_ID_AI_AUTOMATION }}` → Your actual folder ID

**Nodes to update**:
- Upload file (TikTok)
- Upload Instagram Video
- Upload YouTube Video

### 5c. Slack Channel ID

1. In Slack, right-click the channel → Copy channel ID (or get from channel details)
2. In the workflow, find "Send Slack" node
3. Replace `{{ SLACK_CHANNEL_ID }}` → Your actual channel ID

## Step 6: Update Credentials in Nodes

1. For each node that shows a credential placeholder:
   - Click the node
   - Click the credential selector
   - Select the matching credential you created in Step 3

**Credential Mappings**:
- `{{ PLEASE_CONFIGURE_GOOGLE_SHEETS_TRIGGER_OAUTH2 }}` → Google Sheets OAuth2 API
- `{{ PLEASE_CONFIGURE_GOOGLE_SHEETS_OAUTH2 }}` → Google Sheets OAuth2 API
- `{{ PLEASE_CONFIGURE_GOOGLE_DRIVE_OAUTH2 }}` → Google Drive OAuth2 API
- `{{ PLEASE_CONFIGURE_ARCADS_BASIC_AUTH }}` → HTTP Basic Auth (Arcads)
- `{{ PLEASE_CONFIGURE_OPENAI_API }}` → OpenAI API
- `{{ PLEASE_CONFIGURE_SLACK_OAUTH2 }}` → Slack OAuth2 API

**Nodes to update**:
- Google Sheets Trigger
- Read All Rows
- Arcads → Generate Product ID
- Get Upload URL
- Generate TikTok/Instagram/YouTube
- Poll TikTok/Instagram/YouTube
- Upload file (TikTok/Instagram/YouTube)
- Write Output Sheet
- Mark Intake DONE
- Generate Captions
- Send Slack

## Step 7: Test the Workflow

### 7a. Add Test Data
1. Open your Intake Google Sheet
2. Add a test row with:
   - Brand Name: "TestBrand"
   - Product Name: "TestProduct"
   - Tagline: "This is a test"
   - Image URL: A valid image URL (must be publicly accessible)
   - Status: "READY"

### 7b. Manual Execution
1. In n8n, click **Test workflow** or **Execute workflow**
2. Monitor the execution in the **Execution** panel
3. Check for errors in individual node outputs

### 7c. Check Outputs
1. Review Google Drive for uploaded videos
2. Check the Output Sheet for populated data
3. Verify Slack notification was sent

### 7d. Troubleshoot
- **No Google Sheets data**: Check credential permissions
- **API errors**: Verify API keys are valid
- **Video generation fails**: Check Arcads account and API status
- **Missing captions**: Verify OpenAI account has credits

## Step 8: Configure Trigger (Optional)

The workflow is set to trigger on new Google Sheets entries. To customize:

1. Click **Google Sheets Trigger** node
2. Options:
   - **Polling mode** (current): Checks sheet periodically
   - **Webhook mode**: Instant triggers (requires external setup)

To use polling:
1. Check "Poll Times" settings
2. Recommended: Check every 5-10 minutes
3. Click "Save"

## Step 9: Activate Workflow

1. In n8n, click **Activate** (toggle switch, top right)
2. The workflow will now process rows marked as "READY"
3. Monitor the **Executions** tab for activity

## Platform Control

To enable/disable specific platforms:

1. Click **Platform Control** node (in the main flow)
2. Edit the JavaScript code:

```javascript
return [{
  json: {
    ...$json,
    run_tiktok: true,      // true/false
    run_instagram: true,   // true/false
    run_youtube: true      // true/false
  }
}];
```

3. Save changes
4. Re-activate workflow for changes to take effect

## Advanced Configuration

### Video Generation Settings

Edit each platform's generate node to adjust:

**Parameters** (in "Generate TikTok/Instagram/YouTube" nodes):
- `resolution`: "720p" (other options: "480p", "1080p")
- `duration`: 5 (seconds, adjustable)
- `model`: "seedance-2.0" (check Arcads docs for other models)
- `aspectRatio`: Platform-specific (9:16, 1:1, 16:9)

### Polling Timeouts

Adjust wait times if videos take longer to generate:

- **Wait TikTok Init**: Initial wait before polling (seconds)
- **Wait TikTok Retry**: Wait between retry polls (seconds)

Increase if Arcads is slow; decrease for faster feedback.

### Caption Generation

Edit "Generate Captions" node to change AI behavior:

1. Modify the system prompt for different caption styles
2. Adjust temperature/parameters in node settings
3. Change the user prompt to include specific requirements

## Security Best Practices

1. **Never commit credentials** to version control
2. **Use environment variables** for sensitive values in production
3. **Rotate API keys** regularly
4. **Limit OAuth scopes** to minimum required
5. **Monitor n8n logs** for security events
6. **Use separate credentials** for dev/staging/production

## Monitoring & Logs

1. Check **Executions** tab for workflow runs
2. Review individual node outputs for debugging
3. Enable **Logging** in workflow settings for detailed logs
4. Monitor Slack channel for notifications

## Next Steps

1. Test with a few products
2. Monitor output quality
3. Adjust prompts and video settings as needed
4. Scale up batch processing
5. Set up alerting for failures

## Common Issues & Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| Google Sheets credential error | Invalid OAuth token | Re-authenticate in Credentials |
| "Name must be longer than 2 chars" | Missing product_name | Check "Set Variables" node assigns all fields |
| API rate limit | Too many requests | Increase polling interval |
| Video generation timeout | Arcads is slow | Increase wait times |
| Missing Slack notifications | Wrong channel ID | Verify channel ID and bot permissions |
| Low caption quality | Generic AI prompt | Customize system prompt in OpenAI node |

## Support & Debugging

Enable debug mode:
1. Click workflow settings
2. Enable "Disable error front-loading"
3. Click "Logging" tab
4. Set level to "Debug"

Run in test mode to check single nodes without executing full workflow.
