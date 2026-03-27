---
name: boring-youtube-publisher
description: "Upload videos and Shorts to YouTube using Boring. Use when the user says 'upload to YouTube', 'publish YouTube video', 'post a YouTube Short', 'upload video with thumbnail', or wants to upload videos with titles, descriptions, tags, thumbnails, and captions to their YouTube channel."
version: 1.0.0
metadata:
  openclaw:
    emoji: "🎬"
    requires:
      env:
        - BORING_API_KEY
    primaryEnv: BORING_API_KEY
---

# Boring YouTube Publisher

Upload videos and Shorts to YouTube with full metadata support. Powered by [Boring](https://boring-doc.aiagent-me.com).

## Prerequisites

1. **Get your API key** at [boring.aiagent-me.com](https://boring.aiagent-me.com) → Settings → Generate API Key
2. **Add Remote MCP**: Add Boring MCP link `https://boring.aiagent-me.com/mcp` to Claude settings (no local install needed)
3. **Set** `BORING_API_KEY` environment variable
4. **Connect YouTube** at [boring.aiagent-me.com](https://boring.aiagent-me.com) — select the target channel during OAuth

## Workflow

### Step 1: List Accounts

Call `boring_list_accounts` and filter for `youtube` platform.

### Step 2: Prepare Video Content

YouTube requires a video file. Gather from the user:
- **Video file** (required): MP4, MOV, AVI, WMV, FLV — up to 12 hours
- **Title** (required): Max 100 characters
- **Description** (optional): Up to 5,000 characters
- **Tags** (optional): Extracted from hashtags in the description
- **Thumbnail** (optional): JPG/PNG, min 640x360, max 2MB, recommended 1280x720
- **Captions** (optional): SRT or VTT file

### Step 3: Prepare Media URLs

Upload files to get public URLs:
- **Local video**: `boring_upload_file` with `file_path`
- **Video URL**: `boring_upload_from_url` to re-host
- **Google Drive**: Pass directly

The `media_urls` array follows a specific order:
```
media_urls: [
  "https://...video.mp4",           // [0] Video file (required)
  "https://...thumbnail.jpg",       // [1] Custom thumbnail (optional)
  "https://...captions.srt"         // [2] Caption/subtitle file (optional)
]
```

### Step 4: Format the Text Field

YouTube uses a special text format — title and description are separated by a double newline:

```
text: "My Video Title\n\nThis is the video description. It can be up to 5,000 characters.\n\n#tag1 #tag2 #tag3"
```

- First line before `\n\n` = **Title** (max 100 chars)
- Everything after = **Description** (max 5,000 chars)
- Hashtags in description are automatically extracted as **Tags**

### Step 5: Publish

Call `boring_publish_post`:
```
boring_publish_post(
  account_id="<youtube_account_id>",
  platform="youtube",
  text="Video Title\n\nDescription of the video\n\n#shorts #trending",
  media_urls=["https://...video.mp4", "https://...thumb.jpg"]
)
```

**For YouTube Shorts**: Include `#shorts` in the title or description, and use vertical (9:16) video under 60 seconds.

### Step 6: Report

Show:
- Video ID and YouTube URL
- Upload status (processing may take a few minutes on YouTube's side)
- Default visibility is Public

## YouTube-Specific Notes

- **Video required**: YouTube only accepts video uploads (no photo posts)
- **Title max**: 100 characters
- **Description max**: 5,000 characters
- **Thumbnail**: JPG/PNG, 1280x720 recommended, max 2MB
- **Captions**: SRT or VTT format
- **Token**: 1-hour access token with auto-refresh (refresh token never expires)
- **Default visibility**: Public
- **Processing**: After upload, YouTube may take minutes to process the video
- **Permissions**: `youtube.upload`, `youtube.readonly`

## Error Handling

| Error | Solution |
|-------|----------|
| `MediaRequired` | YouTube requires a video file |
| `VideoProcessingFailed` | Check video format (MP4 recommended) or file may be corrupted |
| `MediaTooLarge` | Video file too large |
| `TextTooLong` | Title max 100 chars, description max 5,000 chars |
| `TokenExpired` | Rare — refresh token auto-renews. Reconnect if needed |

## Examples

**Simple video upload**:
```
boring_publish_post(
  account_id="abc",
  platform="youtube",
  text="My Amazing Video\n\nCheck out this cool content!",
  media_urls=["https://example.com/video.mp4"]
)
```

**Full upload with thumbnail and captions**:
```
boring_publish_post(
  account_id="abc",
  platform="youtube",
  text="Tutorial: How to Use Boring\n\nStep-by-step guide to social media automation.\n\n#tutorial #automation #socialmedia",
  media_urls=[
    "https://example.com/tutorial.mp4",
    "https://example.com/thumbnail.jpg",
    "https://example.com/captions.srt"
  ]
)
```

## Documentation

Full API docs: [boring-doc.aiagent-me.com](https://boring-doc.aiagent-me.com)
