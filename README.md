# AI UGC Content Engine - Multi Platform

An n8n-powered automation workflow that generates professional UGC (User Generated Content) videos across multiple platforms (TikTok, Instagram, YouTube) using AI video generation.

## 🎯 Overview

This workflow automates the entire UGC content creation pipeline:

1. **Data Intake** - Reads product information from Google Sheets (brand name, product name, tagline, image URLs)
2. **Image Processing** - Handles multiple product images, uploads them to the Arcads API
3. **Video Generation** - Generates platform-optimized videos:
   - **TikTok** - 9:16 vertical format (5 seconds)
   - **Instagram** - 1:1 square format (5 seconds)
   - **YouTube** - 16:9 widescreen format (5 seconds)
4. **Caption Generation** - Uses OpenAI to create platform-specific captions and hashtags
5. **Output & Notification** - Writes results to Google Sheets and notifies via Slack

## 📋 Features

- **Batch Processing** - Handles multiple product rows from intake sheet
- **Platform-Specific Content** - Generates optimized videos for each platform
- **Smart Image Handling** - Supports multiple images per product, uploads to Arcads
- **Polling & Retry Logic** - Waits for Arcads video generation with exponential backoff
- **Error Handling** - Graceful fallbacks when platforms are disabled
- **Google Drive Integration** - Automatically uploads generated videos to Google Drive
- **Slack Notifications** - Real-time alerts when content is ready

## 🔧 Required Integrations

### APIs & Services

| Service | Purpose | Auth Method |
|---------|---------|-------------|
| **Arcads** | AI Video Generation | HTTP Basic Auth |
| **Google Sheets** | Data Input & Output | OAuth2 |
| **Google Drive** | Video Storage | OAuth2 |
| **OpenAI** | Caption Generation | API Key |
| **Slack** | Notifications | OAuth2 |

### Google Sheets Structure

**Intake Sheet** - Input data:
- `Brand Name` - Brand/company name
- `Product Name` - Product name
- `Tagline` - Product description/tagline
- `Image URL` - URL(s) to product images (comma or newline separated for multiple)
- `Status` - Set to "READY" to trigger processing

**Output Sheet** - Generated content:
- All intake columns plus:
- `Video URL 9:16` - TikTok video link
- `Video URL 1:1` - Instagram video link
- `Video URL 16:9` - YouTube video link
- `TikTok Caption` & `TikTok Hashtags`
- `Instagram Caption` & `Instagram Hashtags`
- `YouTube Caption` & `YouTube Hashtags`
- `Status` - Set to "DONE" when processing completes

## 🚀 Quick Start

### Prerequisites

- n8n instance (self-hosted or cloud)
- Google Workspace account
- Arcads API access
- OpenAI API key
- Slack workspace

### Setup Instructions

See [SETUP.md](SETUP.md) for detailed configuration steps.

### Basic Workflow

1. Add product rows to the Intake Google Sheet
2. Set `Status = READY` on rows you want processed
3. Workflow automatically triggers (or manually execute)
4. Monitor Slack for completion notifications
5. Find videos in Google Drive and output sheet

## 📊 Data Flow

```
Google Sheets (Intake)
    ↓
Filter READY Rows
    ↓
Loop & Set Variables
    ↓
Parse Image URLs
    ↓
Arcads: Generate Product ID
    ↓
Upload Images to Arcads
    ↓
    ├→ Generate TikTok (9:16)
    ├→ Generate Instagram (1:1)
    └→ Generate YouTube (16:9)
    ↓
Poll for Completion
    ↓
Download Videos
    ↓
Upload to Google Drive
    ↓
Generate Captions (OpenAI)
    ↓
Write Output Sheet
    ↓
Mark Intake DONE
    ↓
Slack Notification
```

## ⚙️ Platform Control

Edit the "Platform Control" node to enable/disable platforms:

```javascript
return [{
  json: {
    ...$json,
    run_tiktok: true,      // Generate TikTok video
    run_instagram: false,  // Skip Instagram
    run_youtube: false     // Skip YouTube
  }
}];
```

## 🔐 Security Notes

- All credentials are stored in n8n's secure credential system
- Never commit credentials or API keys to version control
- See [CONFIGURATION.md](CONFIGURATION.md) for secure setup

## 📝 Workflow Nodes

- **Google Sheets Trigger** - Monitors intake sheet for new rows
- **Filter READY Rows** - Processes only rows with Status = READY
- **Loop Over Rows** - Batch processes multiple rows
- **Set Variables** - Extracts and formats product data
- **Parse Image URLs** - Handles multiple images per product
- **Arcads Nodes** - API calls for video generation
- **Google Drive Nodes** - Uploads generated videos
- **OpenAI Node** - Generates captions and hashtags
- **Slack Node** - Sends completion notifications

## 🐛 Troubleshooting

### Video generation fails
- Check Arcads API credentials and status
- Verify image URLs are accessible
- Review polling timeout settings

### Captions not generating
- Verify OpenAI API key and account has credits
- Check JSON parsing in "Parse Captions" node

### Slack notifications not sending
- Verify Slack OAuth token has required scopes
- Confirm channel ID is correct and bot has access

## 📈 Performance Considerations

- Batch size affects processing time (default: 1 row per trigger)
- Arcads generation takes 120 seconds + processing time
- Polling intervals: 120s initial wait, 60s retry
- Multiple images increase processing time

## 🔄 Updates & Maintenance

For updates to prompts or generation settings:
1. Edit the "Set Variables" node for prompts
2. Edit video generation nodes for parameters (resolution, duration, model)
3. Edit "Generate Captions" node for AI system prompt

## 📞 Support

For issues with:
- **n8n workflow** - Check node configuration and error logs
- **Arcads API** - Contact Arcads support
- **Google APIs** - Check Google Cloud Console
- **OpenAI** - Review OpenAI documentation

## 📄 License

MIT License - See [LICENSE](LICENSE)

## 📚 Additional Resources

- [n8n Documentation](https://docs.n8n.io)
- [Arcads API Docs](https://arcads.ai/docs)
- [Google Sheets API](https://developers.google.com/sheets/api)
- [OpenAI API Reference](https://platform.openai.com/docs)
