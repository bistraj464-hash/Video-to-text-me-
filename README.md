# ClipText AI — Video to Text

A full-stack video transcription website. The static frontend is deployed to Netlify; the Python API runs on a separate server with FFmpeg installed.

## Features
- Upload MP4, MOV, MKV, AVI, WEBM and MPEG videos
- Auto-detect speech language or choose a language hint
- Extract audio with FFmpeg and transcribe it with OpenAI Audio API
- Split long audio into smaller segments
- Show segment timestamps in the transcript; copy/download exports clean text without timestamps
- Copy transcript and download TXT
- Save up to 8 recent transcripts in this browser's local storage
- Responsive mobile and desktop layout

## Important requirements
1. An OpenAI API key is required. API usage may incur charges according to your account and model pricing.
2. The API key belongs only in the backend environment variables. Never add it to frontend JavaScript, Netlify public variables, or GitHub.
3. FFmpeg must be installed on the backend. The included Dockerfile installs it.
4. The included backend uses `whisper-1` because it returns segment timestamp metadata. This model is documented as deprecated and scheduled to shut down on February 26, 2027. For a long-lived deployment, switch to the currently recommended `gpt-transcribe` model and update the timestamp UI/implementation as needed. See https://developers.openai.com/api/docs/guides/speech-to-text
5. The current interface displays segment timestamps. Copy and TXT download contain clean transcript text without timestamps.

## Project structure
```text
AI-Video-to-Text/
├── frontend/
│   ├── netlify.toml
│   └── public/
│       ├── index.html
│       ├── styles.css
│       ├── app.js
│       └── config.js
├── backend/
│   ├── main.py
│   ├── requirements.txt
│   ├── Dockerfile
│   ├── render.yaml
│   └── .env.example
├── .gitignore
└── README.md
```

## 1) Deploy the backend
The easiest deployment path is a Docker-capable Python host such as Render.

1. Push this project to a private or public GitHub repository.
2. In Render, create a new **Blueprint** and select the repository.
3. Set the Blueprint root directory to `backend` if prompted; otherwise create a Docker web service using `backend/Dockerfile`.
4. Add `OPENAI_API_KEY` as a secret environment variable in the backend service settings.
5. Set `ALLOWED_ORIGINS` to your Netlify site URL after deploying the frontend (for example `https://your-site.netlify.app`).
6. Deploy and open `https://YOUR-BACKEND-URL/health`. It should return `{"status":"ok","ffmpeg_required":true}`.

Do not commit `.env` or your API key.

## 2) Configure and deploy the frontend to Netlify
1. Open `frontend/public/config.js`.
2. Replace the empty URL with your backend's HTTPS URL:
   ```js
   window.APP_CONFIG = { API_BASE_URL: "https://YOUR-BACKEND-URL" };
   ```
3. In Netlify, import the GitHub repository.
4. Set **Base directory** to `frontend`.
5. Set **Publish directory** to `public`.
6. No build command is needed. Deploy the site.
7. After deploy, update backend `ALLOWED_ORIGINS` to the exact Netlify site URL and redeploy the backend.

## 3) Run locally
Backend (Python 3.11+ and FFmpeg required):
```bash
cd backend
python -m venv .venv
# Windows: .venv\\Scripts\\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
# Set OPENAI_API_KEY in your shell environment first.
uvicorn main:app --reload --port 8000
```
Frontend: serve `frontend/public` with a local static server. Set `API_BASE_URL` in `config.js` to `http://localhost:8000`. Do not open the HTML as a `file://` URL because browser requests and clipboard permissions may not work correctly.

## Limits and troubleshooting
- The frontend rejects files above 500 MB. The backend has its own configurable upload cap (`MAX_UPLOAD_MB`).
- Your hosting provider may impose a lower request size or timeout. Increase plan limits or use direct-to-object-storage uploads for very large videos.
- OpenAI audio uploads have a 25 MB limit for the documented file endpoint; the backend extracts MP3 audio and splits it into 10-minute chunks to stay under that limit in typical cases.
- `FFmpeg is not installed`: deploy using the included Dockerfile.
- `Backend URL is not configured`: set `API_BASE_URL` in `frontend/public/config.js`.
- `CORS`: set `ALLOWED_ORIGINS` to your exact Netlify domain, including `https://`.
- `OPENAI_API_KEY`: add it in the backend host's secret environment settings.
- Transcript history is stored only in the current browser/device. Clearing browser site data may erase it.

## Privacy
Uploaded videos are temporarily written to the backend's temporary directory and removed when processing finishes. The audio is sent to the configured AI transcription provider. Do not upload private videos unless you have permission to process them.
