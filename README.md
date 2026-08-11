# Free AI

A small AI chat application built with HTML, CSS, and JavaScript.

The frontend is hosted on GitHub Pages, authentication and chat history are handled by Supabase, and AI requests go through a Cloudflare Worker so the OpenRouter and Tavily API keys never need to be exposed to the browser.

## Features

* Google sign-in with Supabase Auth
* Persistent conversations
* Multiple chat conversations
* Chat history stored in Supabase
* Row Level Security for user-owned chats and messages
* Streaming AI responses
* OpenRouter integration
* `openrouter/free` model routing
* Optional web search with Tavily
* Markdown rendering
* Code blocks with copy button
* Dark Discord-like interface
* Light/dark theme
* Responsive layout
* Stop generation
* Local session persistence
* GitHub Pages frontend

## Architecture

```text
                         GitHub Pages
                              │
                    ┌─────────┴─────────┐
                    │                   │
                index.html          chat.html
                    │                   │
                    │ Google Login      │
                    ▼                   │
               Supabase Auth            │
                    │                   │
                    │ access token      │
                    └─────────┬─────────┘
                              │
                              ▼
                     Cloudflare Worker
                              │
                ┌─────────────┼─────────────┐
                │             │             │
                ▼             ▼             ▼
            JWT verify      Tavily      OpenRouter
            locally        (optional)     streaming
                │             │             │
                └─────────────┴─────────────┘
                              │
                              ▼
                         AI response
                              │
                              ▼
                         chat.html
                              │
                              ▼
                           Supabase
                         chats/messages
```

The Worker verifies Supabase JWTs locally using the project's JWKS keys instead of making an Auth API request for every message. This reduces an unnecessary network round trip.

## Project structure

```text
open-router-simple-ai/
│
├── index.html
├── chat.html
├── favicon.ico
├── README.md
└── LICENSE
```

The project intentionally uses plain HTML, CSS, and JavaScript. There is no frontend framework or build step.

## Technologies

* HTML5
* CSS3
* JavaScript
* Supabase Auth
* Supabase PostgreSQL
* Supabase Row Level Security
* Cloudflare Workers
* OpenRouter
* Tavily
* Marked.js
* DOMPurify

## Getting started

### 1. Clone the repository

```bash
git clone https://github.com/hosseinb1111/open-router-simple-ai.git
cd open-router-simple-ai
```

### 2. Configure Supabase

Create a Supabase project and run the SQL schema from this project in:

```text
Supabase Dashboard
→ SQL Editor
→ New query
```

The database contains:

```text
profiles
chats
messages
```

Row Level Security is enabled so authenticated users can only access their own conversations.

### 3. Configure Google Login

In Supabase:

```text
Authentication
→ Providers
→ Google
```

Enable Google and provide your Google OAuth Client ID and Client Secret.

For your GitHub Pages application, the JavaScript origin is:

```text
https://hosseinb1111.github.io
```

The Supabase callback URL should be added to your Google OAuth application's authorized redirect URIs.

Your Supabase redirect configuration should also include:

```text
https://hosseinb1111.github.io/open-router-simple-ai/chat.html
```

For local development, you can additionally use:

```text
http://localhost:3000/chat.html
```

## Frontend configuration

Both `index.html` and `chat.html` need your Supabase project information.

Replace:

```javascript
const SUPABASE_URL =
  "YOUR_SUPABASE_URL";

const SUPABASE_PUBLISHABLE_KEY =
  "YOUR_SUPABASE_PUBLISHABLE_KEY";
```

with your actual Supabase values.

In `chat.html`, also set the Cloudflare Worker URL:

```javascript
const WORKER_URL =
  "https://your-worker.your-subdomain.workers.dev";
```

The Supabase publishable/anon key is designed to be usable in client-side applications. Database access is protected by Row Level Security.

Do not put the Supabase `service_role` key in the frontend.

## Cloudflare Worker

The Worker acts as the secure backend between the browser and the AI APIs.

It is responsible for:

* verifying the Supabase user's access token
* validating requests
* calling Tavily when web search is enabled
* calling OpenRouter
* streaming the OpenRouter response back to the browser

The Worker should contain these bindings:

### Variable

```text
SUPABASE_URL
```

Example:

```text
https://your-project.supabase.co
```

### Secrets

```text
OPENROUTER_API_KEY
TAVILY_API_KEY
```

Do not commit these values to GitHub.

Cloudflare Worker Secrets should be used for API keys and other sensitive credentials.

## OpenRouter

The Worker uses:

```javascript
const MODEL = "openrouter/free";
```

This allows OpenRouter to select an available free model instead of locking the application to one specific model.

The Worker enables streaming with:

```javascript
stream: true
```

The frontend reads the Server-Sent Event stream and displays the response as it is generated.

OpenRouter's free model availability and limits can change over time, so `openrouter/free` should not be treated as an unlimited or guaranteed-free service.

## Tavily

Web search is optional.

The user can turn on the search button in the chat interface.

When enabled:

```text
User message
    ↓
Cloudflare Worker
    ↓
Tavily
    ↓
Search results
    ↓
OpenRouter
    ↓
Streaming answer
```

Tavily is only called when web search is enabled.

The Tavily API key stays inside the Cloudflare Worker.

## Security

The application is designed so that the most important API credentials never reach the browser.

### Safe to expose in frontend

```text
SUPABASE_URL
SUPABASE_PUBLISHABLE_KEY
```

The Supabase publishable key is not intended to be a secret. Access to your database is controlled using Row Level Security.

### Keep private

```text
OPENROUTER_API_KEY
TAVILY_API_KEY
SUPABASE_SERVICE_ROLE_KEY
Google OAuth Client Secret
```

These should never be committed to the repository or placed in frontend JavaScript.

## Database

The main database tables are:

### `profiles`

Stores basic information associated with authenticated users.

```text
id
email
full_name
avatar_url
created_at
updated_at
```

### `chats`

Stores individual conversations.

```text
id
user_id
title
created_at
updated_at
```

### `messages`

Stores messages belonging to each conversation.

```text
id
chat_id
role
content
created_at
```

The relationship is:

```text
User
 │
 └── chats
       │
       └── messages
```

Row Level Security ensures that users can only access their own chats and the messages inside those chats.

## Local development

You can run the project using a simple local HTTP server.

For example, with Python:

```bash
python -m http.server 3000
```

Then open:

```text
http://localhost:3000
```

Google OAuth does not work reliably when opening the HTML files directly with `file://`, so use an HTTP server during development.

Add your local callback URL to Supabase:

```text
http://localhost:3000/chat.html
```

and configure the appropriate Google OAuth settings.

## Deployment

The frontend can be deployed to GitHub Pages.

For this repository, the production website is:

```text
https://hosseinb1111.github.io/open-router-simple-ai/
```

After pushing changes to the repository, GitHub Pages can serve the updated static files.

The Cloudflare Worker is deployed separately from the frontend.

## Environment variables and secrets

Do not create a public `.env` file containing your real API keys.

For this project:

```text
Frontend
└── public Supabase configuration

Cloudflare Worker
├── SUPABASE_URL
├── OPENROUTER_API_KEY
└── TAVILY_API_KEY
```

If you use GitHub Actions later, GitHub repository secrets can be used by the workflow, but they should not be injected into frontend JavaScript because anything shipped to the browser is public.

## Known limitations

The application currently depends on free AI and search services, so availability and limits can vary.

OpenRouter's free routing can have:

* lower rate limits
* slower response times
* changing model availability
* temporary provider congestion

Tavily usage is also subject to the limits of the selected Tavily plan.

The Cloudflare Worker currently provides the security boundary for API credentials, but additional rate limiting can be added later if the application gets significant traffic.

## License

This project is licensed under the MIT License.

See the `LICENSE` file for details.

## Disclaimer

This project is not affiliated with OpenRouter, Supabase, Cloudflare, Tavily, Google, OpenAI, Meta, or any individual AI model provider.

AI-generated information can be inaccurate. Verify important information before relying on it.

---

Built as a personal project for experimenting with AI APIs, authentication, web search, and modern web development.
