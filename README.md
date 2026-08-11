# Free AI

A simple AI chat interface built with plain HTML, CSS, and JavaScript.

The project uses **OpenRouter** to connect to different AI models through a single API. The interface is designed to feel more like a simple Discord-style chat rather than a heavily polished commercial AI application.

## Features

* Dark Discord-like interface
* OpenRouter API integration
* Support for OpenRouter-compatible models
* Markdown rendering
* Code block formatting
* Copy code button
* Copy AI responses
* Conversation history using `localStorage`
* Clear conversation button
* Stop generation
* Typing indicator
* Suggested prompts
* Responsive design for desktop and mobile
* Light/dark theme toggle
* Automatic textarea resizing
* `Enter` to send
* `Shift + Enter` for a new line
* Basic HTML sanitization with DOMPurify

## Technologies

This project is intentionally simple and doesn't require a framework.

* HTML
* CSS
* JavaScript
* OpenRouter API
* Marked.js
* DOMPurify

External libraries are loaded through CDN links.

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
cd YOUR_REPOSITORY
```

Or download the repository as a ZIP file and extract it.

### 2. Get an OpenRouter API key

Create an account on OpenRouter and generate an API key from your account dashboard.

You can find OpenRouter here:

https://openrouter.ai/

### 3. Add your API key

Open the JavaScript section of `index.html`.

You'll find something similar to:

```javascript
const API_KEY = "YOUR_OPENROUTER_API_KEY";
```

Replace it with your API key:

```javascript
const API_KEY = "sk-or-v1-xxxxxxxxxxxxxxxx";
```

### 4. Select a model

The model is configured with:

```javascript
const MODEL = "meta-llama/llama-3.3-70b-instruct";
```

You can replace this with another model supported by OpenRouter.

For example:

```javascript
const MODEL = "openai/gpt-oss-120b";
```

or:

```javascript
const MODEL = "google/gemini-2.5-flash";
```

Check OpenRouter's model list for the currently available models.

## Running the project

Because this is a simple static website, you don't need Node.js or a complicated build system.

You can use a local development server.

For example, with Python:

```bash
python -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

You can also use VS Code's Live Server extension or deploy the project to a static hosting service.

## OpenRouter configuration

The application sends requests to:

```javascript
const API_URL = "https://openrouter.ai/api/v1/chat/completions";
```

The request follows OpenRouter's OpenAI-compatible chat completion format.

A simplified request looks like this:

```javascript
const response = await fetch(API_URL, {
  method: "POST",

  headers: {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${API_KEY}`
  },

  body: JSON.stringify({
    model: MODEL,
    messages: [
      {
        role: "system",
        content: SYSTEM_PROMPT
      },
      ...conversation
    ],
    temperature: 0.7,
    max_tokens: 4096
  })
});
```

## Important security warning

**Do not put a real OpenRouter API key inside a public website.**

If you put the API key directly into `index.html`, anyone who opens the website can potentially inspect the JavaScript and obtain the key.

For example, this is **not safe for a public production website**:

```javascript
const API_KEY = "sk-or-v1-your-real-key";
```

It is okay for local experimentation, but you should use a backend or serverless function for a public deployment.

A safer architecture is:

```text
Browser
   │
   ▼
Your Backend / Serverless Function
   │
   ▼
OpenRouter
   │
   ▼
AI Model
```

The OpenRouter API key stays on the server instead of being exposed to the browser.

## Conversation history

The application stores conversations locally using the browser's `localStorage`.

The storage key is:

```javascript
const STORAGE_KEY = "free-ai-conversation";
```

This means the conversation is stored in the user's browser rather than in a database.

Clearing the conversation removes the stored history.

## Themes

The interface supports both dark and light themes.

The selected theme is stored locally:

```javascript
const THEME_KEY = "free-ai-theme";
```

The application automatically checks the user's system preference when no theme has previously been selected.

## Project structure

The project can remain very small:

```text
free-ai/
│
├── index.html
├── README.md
└── .gitignore
```

If you later move the JavaScript and CSS into separate files, the structure could become:

```text
free-ai/
│
├── index.html
├── css/
│   └── style.css
├── js/
│   └── app.js
├── README.md
└── .gitignore
```

## `.gitignore`

If you are keeping the API key in a local configuration file, make sure that file isn't committed to Git.

For example:

```gitignore
.env
.env.local

node_modules/

.DS_Store
Thumbs.db
```

However, if the API key is written directly inside `index.html`, `.gitignore` will **not** protect it.

Git will still track `index.html`.

For a public GitHub repository, never commit your real API key.

## Customizing the project

Most of the interface can be customized directly through the CSS variables near the beginning of the stylesheet.

For example:

```css
:root {
  --accent: #5865f2;
  --bg: #08090d;
  --surface-solid: #111318;
  --text: #f4f5f7;
}
```

You can change these values to create your own color scheme.

The model can be changed through:

```javascript
const MODEL = "your-model-id";
```

The system prompt can be changed through:

```javascript
const SYSTEM_PROMPT = `
You are Free AI, a helpful AI assistant.
`;
```

## Disclaimer

This project is not affiliated with OpenRouter, Discord, Meta, OpenAI, Google, or any of the AI model providers used through OpenRouter.

The AI can produce incorrect or outdated information. Important information should always be verified.

## License

This project is available under the MIT License.

See the `LICENSE` file for the full license text.

---

Made for experimenting with AI APIs and building things on the web.
