# 📹 Auto YouTube Shorts Publisher — n8n Workflow

> Fully automated pipeline that generates viral science video scripts using AI, renders them into short videos, and publishes them to YouTube — every 3 hours, hands-free.

---

## 🔄 What It Does

```
Schedule (every 3h)
       ↓
Fetch available music tags
       ↓
AI generates 4-scene science video script (Mistral)
       ↓
AI picks the best music tag for the video
       ↓
Send scenes to video rendering API
       ↓
Poll until video is ready
       ↓
Download rendered MP4
       ↓
Upload to YouTube Shorts
```

---

## 🧩 Workflow Nodes

| Node | Type | Purpose |
|------|------|---------|
| Schedule Trigger | Trigger | Fires every 3 hours automatically |
| Configure | Set | Sets the video rendering server URL |
| Get music tags | HTTP Request | Fetches available music tags from the API |
| Group the music tags | Aggregate | Combines all tags into one item |
| Generate content | LLM Chain | Generates 4-scene video script via Mistral |
| Pick the right music | LLM Chain | Selects the best music tag for the video |
| Start generating the video | HTTP Request | Sends scenes to video rendering API |
| Wait | Wait | 60 second pause before checking status |
| Check video status | HTTP Request | Polls the API for render completion |
| Ready? | IF | Routes: ready → download, not ready → wait again |
| Fast Download (wget) | Execute Command | Downloads the rendered MP4 to `/tmp/` |
| Read Video File | Read Binary File | Reads the MP4 into n8n memory |
| Code in JavaScript | Code | Forces MIME type to `video/mp4` |
| YouTube | YouTube | Uploads the video to YouTube |

---

## 🤖 AI Content Generation

The script generation prompt instructs Mistral to create **4-scene viral science videos** covering topics like:

- Astrophysics & cosmology
- Quantum mechanics
- Strange biology & evolution
- Human body & neuroscience
- Deep sea creatures, genetics, psychology

Each scene contains:
- **Text** — spoken narration (1-2 punchy sentences)
- **Search Terms** — concrete, filmable nouns for Pexels stock footage (e.g. `galaxy`, `microscope`, `jellyfish` — NOT abstract words like `mystery` or `discovery`)

---

## ⚙️ Setup

### Prerequisites
- n8n instance running (self-hosted or cloud)
- A video rendering server running at `http://host.docker.internal:3123` (e.g. [Creatomate](https://creatomate.com/) or a custom renderer)
- Mistral Cloud API key
- YouTube OAuth2 credentials connected in n8n

### Credentials Needed

| Service | Credential Type |
|---------|----------------|
| Mistral Cloud | `mistralCloudApi` |
| YouTube | `youTubeOAuth2Api` |

### Configuration

In the **Configure** node, set:
```
SERVER_URL = http://host.docker.internal:3123
```
Change this to your video rendering server's URL.

### YouTube Upload Settings (currently configured)
- Region: `EG` (Egypt) — change to your region code
- Category: `22` (People & Blogs)
- Tags: auto-generated from video title

---

## 🔁 Retry Logic

- **Generate content** and **Pick the right music** both have `retryOnFail: true`
- **Check video status** retries on failure and continues on error
- The **Ready?** loop keeps polling every 60 seconds until the video is rendered

---

## 📁 Files

```
├── video-publisher.json    # n8n workflow export (import this into n8n)
└── README.md
```

### How to Import
1. Open your n8n instance
2. Go to **Workflows → Import from File**
3. Select `video-publisher.json`
4. Connect your credentials
5. Update the `SERVER_URL` in the Configure node
6. Activate the workflow
