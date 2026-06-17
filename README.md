# AI Burp Payload Assistant

Manual AI-assisted reflected XSS payload suggestions for Burp Suite Repeater.

This project is an MVP security testing helper. It is designed for authorized manual penetration testing only. It does not crawl, scan, exploit, modify requests, or send attacks automatically.

## Current Architecture

```text
Burp Suite Repeater
        |
        | right-click "Suggest Payloads"
        v
Burp Extension, Java + Montoya + Swing
        |
        | POST http://127.0.0.1:8000/suggest
        v
FastAPI Backend
        |
        | local deterministic analysis
        v
SecurityContext
        |
        | compact prompt
        v
Google Gemini 2.5 Flash
        |
        | JSON payload suggestions
        v
Burp "AI Payloads" table
```

## What It Can Do

- Add a Burp right-click menu item named `Suggest Payloads`.
- Extract request context from Burp Repeater.
- Extract response headers and response body when available.
- Exclude sensitive headers such as `Cookie`, `Set-Cookie`, `Authorization`, `Proxy-Authorization`, and `X-Api-Key`.
- Send structured JSON to the local FastAPI backend.
- Detect reflected marker positions locally.
- Classify reflected XSS context locally:
  - `HTML_BODY`
  - `HTML_ATTRIBUTE`
  - `SCRIPT_BLOCK`
  - `JS_STRING`
  - `URL_CONTEXT`
  - `JSON_CONTEXT`
  - `UNKNOWN`
- Detect basic encoding behavior:
  - `NONE`
  - `HTML_ENTITY`
  - `URL_ENCODING`
  - `UNICODE_ESCAPE`
  - `PARTIAL_ESCAPE`
  - `UNKNOWN`
- Extract CSP signals from response headers.
- Fingerprint basic WAF/CDN indicators:
  - Cloudflare
  - Akamai
  - Imperva
  - AWS WAF
- Build a compact security context for Gemini.
- Ask Gemini for up to 3 reflected XSS payload suggestions.
- Display payload, reason, and confidence inside Burp.
- Copy payloads from the Swing table.
- Log Gemini token usage in the backend terminal.
- Cache payload suggestions in memory by hostname, context, encoding, and WAF.

## What It Does Not Do

- No autonomous scanning.
- No crawling.
- No automatic exploitation.
- No automatic request sending.
- No automatic request modification.
- No browser automation.
- No stored XSS workflow.
- No DOM XSS analysis.
- No SQLi, SSTI, SSRF, command injection, or other vulnerability classes.
- No persistence or database storage.
- No LangChain, RAG, agents, or vector database.

## Project Layout

```text
.
├── backend
│   ├── app
│   │   ├── analysis
│   │   │   ├── classification
│   │   │   ├── csp
│   │   │   ├── encoding
│   │   │   ├── reflection
│   │   │   └── waf
│   │   ├── api
│   │   ├── core
│   │   ├── models
│   │   └── services
│   ├── tests
│   ├── requirements.txt
│   └── README.md
├── burp-extension
│   ├── src/main/java/com/pentest/aiburp
│   ├── lib/montoya-api-2026.4.jar
│   ├── build.gradle
│   ├── settings.gradle
│   └── README.md
├── docs
│   └── phase-2-context-analysis-architecture.md
└── README.md
```

## Requirements

- Windows PowerShell
- Python 3.11+
- JDK 17+
- Burp Suite with Montoya API support
- Google Gemini API key

The current local JDK path used during development was:

```text
C:\Program Files\Java\jdk-26.0.1
```

## Backend Setup

From the project root:

```powershell
cd C:\Users\akash\Documents\Codex\2026-05-15\i-need-to-develop-a-new\backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Create or update this file:

```text
C:\Users\akash\Documents\Codex\2026-05-15\i-need-to-develop-a-new\backend\.env
```

Add your Gemini API key:

```env
GEMINI_API_KEY=your-google-ai-studio-key-here
GEMINI_MODEL=gemini-2.5-flash
```

Do not commit `.env`. It is intentionally ignored by `.gitignore`.

## Run The Backend

```powershell
cd C:\Users\akash\Documents\Codex\2026-05-15\i-need-to-develop-a-new\backend
.\.venv\Scripts\Activate.ps1
python -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

Expected output:

```text
Uvicorn running on http://127.0.0.1:8000
Application startup complete.
```

Health check:

```powershell
Invoke-RestMethod http://127.0.0.1:8000/health
```

Expected response:

```json
{
  "status": "ok"
}
```

## Build The Burp Extension JAR

If Gradle is available:

```powershell
cd C:\Users\akash\Documents\Codex\2026-05-15\i-need-to-develop-a-new\burp-extension
gradle clean build
```

If Gradle is not available, use the manual JDK build:

```powershell
cd C:\Users\akash\Documents\Codex\2026-05-15\i-need-to-develop-a-new\burp-extension
$sources = Get-ChildItem src\main\java -Recurse -Filter *.java | ForEach-Object { $_.FullName }
& 'C:\Program Files\Java\jdk-26.0.1\bin\javac.exe' --release 17 -cp lib\montoya-api-2026.4.jar -d build\classes\java\main $sources
& 'C:\Program Files\Java\jdk-26.0.1\bin\jar.exe' --create --file build\libs\ai-burp-payload-assistant-0.1.0.jar -C build\classes\java\main .
```

The JAR is created here:

```text
C:\Users\akash\Documents\Codex\2026-05-15\i-need-to-develop-a-new\burp-extension\build\libs\ai-burp-payload-assistant-0.1.0.jar
```

Note: JDK 26 may print a Montoya JAR `AccessDeniedException` warning after compilation. In this setup, the compiler still exited successfully and produced usable classes.

## Load The Extension In Burp

1. Open Burp Suite.
2. Go to `Extensions`.
3. Remove the old version of the extension if it is already loaded.
4. Click `Add`.
5. Choose extension type `Java`.
6. Select:

```text
C:\Users\akash\Documents\Codex\2026-05-15\i-need-to-develop-a-new\burp-extension\build\libs\ai-burp-payload-assistant-0.1.0.jar
```

The extension name should appear as:

```text
AI Burp Payload Assistant
```

The Burp tab should appear as:

```text
AI Payloads
```

## Use The Tool

1. Start the backend.
2. Load the extension in Burp.
3. Open a request in Repeater.
4. Use a marker value such as:

```text
XAIREF123
```

5. Send the request manually in Repeater.
6. Confirm the response reflects the marker or selected value.
7. Right-click the request.
8. Click `Suggest Payloads`.
9. View payloads in the `AI Payloads` tab.

The table shows:

- Payload
- Reason
- Confidence

You can select a row and copy the payload.

## Backend API

Endpoint:

```text
POST /suggest
```

Example request:

```json
{
  "method": "GET",
  "url": "https://example.test/search?q=XAIREF123",
  "headers": {
    "Content-Type": "text/html"
  },
  "body": null,
  "selected_text": "XAIREF123",
  "parameter_name": "q",
  "content_type": "text/html",
  "response_headers": {
    "Content-Type": "text/html",
    "Content-Security-Policy": "script-src 'self'"
  },
  "response_body": "<html><body>XAIREF123</body></html>",
  "reflection_marker": "XAIREF123"
}
```

Example response:

```json
{
  "payloads": [
    {
      "payload": "<svg/onload=alert(1)>",
      "reason": "HTML body vector",
      "confidence": 0.82
    }
  ]
}
```

## Local Analysis Flow

The backend performs these deterministic steps before calling Gemini:

```text
Response body
  -> reflection detection
  -> reflection position mapping
  -> context classification
  -> encoding detection
  -> CSP extraction
  -> WAF fingerprinting
  -> compact SecurityContext
  -> compact Gemini prompt
```

Gemini receives compact context and strategy hints, not full raw traffic.

Example compact context:

```json
{
  "r": true,
  "ctx": "HTML_ATTRIBUTE",
  "enc": "NONE",
  "waf": "cloudflare",
  "csp_inline": false,
  "nonce": true,
  "strict_dynamic": false,
  "h": ["attr_breakout", "event_handler", "cloudflare_bypass"]
}
```

## Run Tests

Backend tests:

```powershell
cd C:\Users\akash\Documents\Codex\2026-05-15\i-need-to-develop-a-new\backend
.\.venv\Scripts\python.exe -m unittest discover -s tests
```

Last known test result:

```text
Ran 58 tests
OK
```

## Troubleshooting

### `GEMINI_API_KEY is not set`

Check:

```text
backend\.env
```

It should contain:

```env
GEMINI_API_KEY=your-google-ai-studio-key-here
```

Restart the backend after changing `.env`.

### `API key not valid`

Generate a new API key in Google AI Studio, update `backend\.env`, and restart Uvicorn.

### `models/gemini-1.5-flash is not found`

Use:

```env
GEMINI_MODEL=gemini-2.5-flash
```

### Burp shows HTTP 422

The extension sent malformed or incomplete JSON. Make sure you are using the latest rebuilt JAR.

### Burp shows HTTP 502

The backend reached Gemini but Gemini failed, returned invalid output, or the parser rejected the response. Check the backend PowerShell logs for details.

### Payloads are generic

Make sure the response contains the selected marker or `XAIREF123`. If no reflection is detected, the backend may classify context as `UNKNOWN`.

### No response context is detected

Send the request manually in Repeater first, then right-click after Burp has a response.

## Security Notes

- Do not store API keys in source code.
- Do not commit `backend\.env`.
- The extension strips common sensitive headers before sending data to the backend.
- The backend runs locally on `127.0.0.1:8000`.
- This is a manual testing assistant, not an autonomous attack tool.

## Current Limitations

- Reflected XSS only.
- Requires a reflected marker or selected value for best results.
- Context classification is deterministic but lightweight.
- CSP and WAF detection are basic signal extractors, not complete scanners.
- Payload cache is in-memory only.
- Gemini output quality can vary.
- No persistent project state.
- No automatic verification that a payload executes.

## Useful Paths

Backend:

```text
C:\Users\akash\Documents\Codex\2026-05-15\i-need-to-develop-a-new\backend
```

Burp extension:

```text
C:\Users\akash\Documents\Codex\2026-05-15\i-need-to-develop-a-new\burp-extension
```

Extension JAR:

```text
C:\Users\akash\Documents\Codex\2026-05-15\i-need-to-develop-a-new\burp-extension\build\libs\ai-burp-payload-assistant-0.1.0.jar
```

Architecture notes:

```text
C:\Users\akash\Documents\Codex\2026-05-15\i-need-to-develop-a-new\docs\phase-2-context-analysis-architecture.md
```
** Future enhancements include browser-based exploit verification, authentication-aware replay workflows, multi-user collaboration, advanced attack path generation, AI-assisted threat modeling, vulnerability correlation across multiple requests, and deeper business logic vulnerability analysis. Planned improvements also include support for distributed storage, integration with ticketing and vulnerability management platforms, expanded exploit validation capabilities, and enhanced contextual AI guidance for complex application architectures. The long-term vision is to evolve the platform into a comprehensive AI-assisted offensive security workspace that combines deterministic analysis, exploit verification, and intelligent testing recommendations across multiple vulnerability classes, including XSS, SQL Injection, SSRF, and Command Injection.
