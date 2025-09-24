## AI Meeting Hero

A single-page web app to turn meeting recordings into actionable knowledge using prompt templates. Frontend is static (Tailwind via CDN). Data is stored in Google Sheets via a Google Apps Script (GAS) web endpoint.

### Live site (GitHub Pages)
- After enabling GitHub Pages for this repo, your site will be available at:
  - `https://<your-github-username>.github.io/ai-meeting-hero/`

### Project structure
- `index.html` — the entire frontend (UI, logic, and API calls)
- `AI_Prompts_Backend - Prompts.csv` — sample data format for prompts (for reference)
- `app_script.txt` — the GAS backend source (copy into Apps Script)

### Local development
1. Serve locally:
   - `python3 -m http.server 8000`
   - Open `http://localhost:8000`
2. Edit `index.html` and refresh. No build step required.

### Deploy to GitHub Pages
1. Ensure `index.html` is at the repo root (already done).
2. Push to `main`:
   - `git add . && git commit -m "deploy" && git push origin main`
3. In GitHub → Repository → Settings → Pages:
   - Source: Deploy from a branch
   - Branch: `main` and Folder: `/ (root)` → Save
4. Wait 1–2 minutes for the `pages build and deployment` workflow to finish.

### Backend (Google Apps Script + Google Sheets)
This app expects a public (or org-restricted) GAS endpoint returning JSON.

- Frontend config in `index.html`:
  - `GOOGLE_SHEET_API_URL = "https://script.google.com/macros/s/<DEPLOYMENT_ID>/exec"`
- Backend code:
  - Open Google Drive → New → Google Apps Script
  - Paste the contents of `app_script.txt`
  - Update `SPREADSHEET_ID` and `SHEET_NAME` accordingly
  - IMPORTANT: Return pure JSON only. Use:
    - `ContentService.createTextOutput(JSON.stringify(obj)).setMimeType(ContentService.MimeType.JSON)`
    - Do NOT use `withHeaders()` (frontend already warns if detected)
  - Deploy → Manage deployments → New deployment → Web app:
    - Execute as: Me
    - Who has access: Anyone (or Only within your org if you control access via Google Workspace)
  - Copy the web app URL into `GOOGLE_SHEET_API_URL`

#### Expected responses
- GET `.../exec?action=getVersion` → `{ status: "success", version: "<SCRIPT_VERSION>" }`
- GET `.../exec` → Array of prompt rows `{ id, title, category, author, content, output, copyCount }`
- POST `.../exec` with body `{ action: 'addPrompt', data }` → `{ status: 'success', newData }`
- POST `.../exec` with body `{ action: 'incrementCopyCount', id }` → `{ status: 'success', newCount }`

### Common issues
- "withHeaders is not a function": Remove any `.withHeaders(...)` usage in GAS; return JSON as shown above.
- CORS / fetch failed: Ensure your GAS deployment access is set correctly and you re-deployed a new version.
- Data types: Ensure `id` and `copyCount` are numeric in responses; the frontend parses them.

### Optional: Access control options
- Fastest no-code gate: Cloudflare Access in front of your Pages/custom domain (Google SSO; restrict to `@yourcompany.com`).
- All-in-Google: Host the page via GAS HTML Service and deploy to "Only within your org".
- App-level auth: Add Google Identity Services to the frontend and protect the backend with token checks.

### License
Internal use. Add your preferred license here if needed.


