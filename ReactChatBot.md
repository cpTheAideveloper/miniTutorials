# Beginner-Friendly Guide: Build a Simple AI Chatbot with React & OpenAI

## What You'll Create
A simple chatbot that uses OpenAI's new Responses API - letting you see both the AI's answer AND its reasoning process!

## Why This Is Cool
- **Super Simple**: Just 10 lines of React code to get a working AI chatbot
- **Shows AI Thinking**: You can see how the AI reasons through answers with the new `reasoning_summary` feature
- **Future-Ready**: Uses OpenAI's newest API that combines chat capabilities with tool-using abilities

## What You Need Before Starting
- Basic knowledge of JavaScript
- Node.js installed on your computer
- A text editor (like VS Code)
- An OpenAI account with an API key

## Step-by-Step Instructions

### Step 1: Create a New React Project
Open your terminal/command prompt and run:
```bash
npm create vite@latest ai-chatbot -- --template react
```
Then navigate to your new project folder:
```bash
cd ai-chatbot
```

### Step 2: Install the OpenAI Package
In your project folder, run:
```bash
npm i openai
```
This installs the OpenAI JavaScript library.

### Step 3: Add Your API Key
1. Create a new file called `.env` in your project's root folder
2. Add this line to the file (replace with your actual API key):
```
VITE_OPENAI_KEY=sk-...
```
> ⚠️ **Security Tip**: Never share your API key or commit it to public repositories!

### Step 4: Create Your Chat Component
Create a new file called `Chat.jsx` in the `src` folder and paste this code:

```jsx
import { useState } from "react";

// Function that calls the OpenAI API
const chat = q => fetch("https://api.openai.com/v1/responses", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${import.meta.env.VITE_OPENAI_KEY}`,
  },
  body: JSON.stringify({
    model: "o3", // OpenAI's reasoning-capable model
    messages: [{ role: "user", content: q }],
    reasoning_summary: "concise", // This shows the AI's thinking process
  }),
}).then(r => r.json());

// The Chat component
export default function Chat() {
  const [msg, setMsg] = useState("");       // Stores current message being typed
  const [log, setLog] = useState([]);       // Stores chat history

  return (
    <>
      {/* Display chat history */}
      <pre>{log.join("\n")}</pre>
      
      {/* Input box for user messages */}
      <input
        value={msg}
        onChange={e => setMsg(e.target.value)}
        onKeyDown={e => {
          if (e.key === "Enter") {
            chat(msg)
              .then(r => setLog(l => [
                ...l,
                "🧑 " + msg,                              // User message
                "🤖 " + r.choices[0].message.content,     // AI response
                "🧩 " + r.reasoning_summary                // AI's reasoning
              ]));
            setMsg("");  // Clear input after sending
          }
        }}
        placeholder="Type a message and press Enter"
      />
    </>
  );
}
```

### Step 5: Update App.jsx to Use Your Chat Component
Open `App.jsx` and replace its contents with:

```jsx
import Chat from './Chat'

function App() {
  return (
    <div className="App">
      <h1>My AI Chatbot</h1>
      <Chat />
    </div>
  )
}

export default App
```

### Step 6: Run Your App
In your terminal, run:
```bash
npm run dev
```

Open the URL shown in your terminal (usually http://localhost:5173) to see your chatbot!

## Try It Out!
Type a question in the input box and press Enter. You'll see:
- Your question
- The AI's answer
- A brief explanation of how the AI reasoned through its answer

## How to Enhance Your Chatbot

### Show More Detailed Reasoning
Change `reasoning_summary: "concise"` to `reasoning_summary: "detailed"` for more in-depth explanations.

### Add Web Search Capability
Extend the API call with a `tools` parameter to let your AI search the web:

```jsx
body: JSON.stringify({
  model: "o3",
  messages: [{ role: "user", content: q }],
  reasoning_summary: "concise",
  tools: [{ type: "web_search" }]  // Add this line
}),
```

### Add Real-Time Streaming
For a more interactive experience, replace the fetch with streaming capabilities so the AI's response appears letter by letter.

## Troubleshooting Tips
- If you see "API key invalid" errors, double-check your key in the `.env` file
- Make sure Vite can access environment variables (they must start with `VITE_`)
- If you see CORS errors, you might need to set up a backend proxy

## Learn More
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [React Documentation](https://react.dev)
- [Vite Documentation](https://vitejs.dev/guide)
