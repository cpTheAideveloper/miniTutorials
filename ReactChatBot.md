# Beginner-Friendly Guide: Build a Simple AI Chatbot with React & OpenAI

## What You'll Create
A simple chatbot that uses OpenAI's new Responses API - letting you see both the AI's answer AND its reasoning process!

![Example of the finished chatbot with UI](https://i.imgur.com/placeholder.jpg)

## Why This Is Cool
- **Beginner-Friendly**: Error-proof code with helpful comments
- **Shows AI Thinking**: See how the AI reasons through answers with the `reasoning_summary` feature
- **Good Looking UI**: Includes simple styling so your chatbot looks professional

## What You Need Before Starting
- Basic knowledge of JavaScript
- Node.js installed on your computer (download from [nodejs.org](https://nodejs.org))
- A text editor (like [VS Code](https://code.visualstudio.com/))
- An OpenAI account with an API key ([Get one here](https://platform.openai.com/signup))

## Step-by-Step Instructions (With Pictures!)

### Step 1: Create a New React Project
Open your terminal/command prompt (search for "Terminal" or "Command Prompt" on your computer).

Type this command and press Enter:
```bash
npm create vite@latest ai-chatbot -- --template react
```

You'll see something like this:
![Terminal showing project creation](https://i.imgur.com/placeholder1.jpg)

When it finishes, type this command to enter your new project folder:
```bash
cd ai-chatbot
```

### Step 2: Install Required Packages
In your project folder, run:
```bash
npm install
```
This installs the basic React packages.

Then install the OpenAI package:
```bash
npm i openai
```

### Step 3: Add Your API Key
1. Create a new file called `.env` in your project's main folder
   - In VS Code: Right-click in the file explorer > New File > name it `.env`
   - In other editors: Create a new empty file and save it as `.env`

2. Add this line to the file (replace with your actual API key from OpenAI):
```
VITE_OPENAI_KEY=sk-...
```

> ⚠️ **Security Warning**: Your API key is like a password! Never share it, post it online, or include it in code you share with others.

### Step 4: Create Your Chat Component
Create a new file called `Chat.jsx` in the `src` folder and paste this code:

```jsx
import { useState } from "react";

// Function that calls the OpenAI API with the updated response format
const chat = async (userMessage) => {
  try {
    // Make API request to OpenAI
    const response = await fetch("https://api.openai.com/v1/responses", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${import.meta.env.VITE_OPENAI_KEY}`,
      },
      body: JSON.stringify({
        model: "o3", // OpenAI's reasoning-capable model
        input: [{ role: "user", content: userMessage }], // Changed from 'messages' to 'input'
        reasoning: { effort: "medium" }, // Changed from 'reasoning_summary' to 'reasoning'
      }),
    });
    
    // Convert the response to JSON
    const data = await response.json();
    
    // Log the response for debugging
    console.log("API Response:", data);
    
    // Check if we have the expected data structure based on the new format
    if (!data || !data.output) {
      console.error("Unexpected API response format:", data);
      return {
        message: "Sorry, I received an unexpected response format. Please try again.",
        reasoning: "Error processing response"
      };
    }
    
    // Extract the message from the new response structure
    let message = "No response found";
    let reasoning = "No reasoning available";
    
    // Find the message output in the array
    const messageOutput = data.output.find(item => item.type === "message");
    if (messageOutput && messageOutput.content && messageOutput.content[0] && messageOutput.content[0].text) {
      message = messageOutput.content[0].text;
    }
    
    // Find reasoning if available
    const reasoningOutput = data.output.find(item => item.type === "reasoning");
    if (reasoningOutput && reasoningOutput.summary) {
      reasoning = Array.isArray(reasoningOutput.summary) 
        ? reasoningOutput.summary.join("\n") 
        : reasoningOutput.summary || "AI thought about this.";
    } else if (data.reasoning && data.reasoning.summary) {
      reasoning = data.reasoning.summary || "AI thought about this.";
    }
    
    // Return the extracted data
    return {
      message: message,
      reasoning: reasoning
    };
  } catch (error) {
    // Handle any errors
    console.error("Error calling OpenAI API:", error);
    return {
      message: "Sorry, there was an error communicating with the AI service.",
      reasoning: "Error calling API"
    };
  }
};

// The Chat component
export default function Chat() {
  // State to store the current message being typed
  const [msg, setMsg] = useState("");
  
  // State to store our chat history
  const [log, setLog] = useState([]);
  
  // State to track if we're currently waiting for a response
  const [isLoading, setIsLoading] = useState(false);

  // Handle sending a message
  const handleSendMessage = async () => {
    // Don't do anything if the message is empty
    if (!msg.trim()) return;
    
    // Add user message to chat log immediately
    const userMessage = msg;
    setLog(prevLog => [...prevLog, `🧑 ${userMessage}`]);
    
    // Clear input field
    setMsg("");
    
    // Show loading state
    setIsLoading(true);
    
    try {
      // Call the OpenAI API
      const result = await chat(userMessage);
      
      // Add AI response to chat log
      setLog(prevLog => [
        ...prevLog,
        `🤖 ${result.message}`,
        `🧩 ${result.reasoning}`
      ]);
    } catch (error) {
      // Handle any errors
      console.error("Error in handleSendMessage:", error);
      setLog(prevLog => [
        ...prevLog,
        "❌ Sorry, something went wrong. Please try again."
      ]);
    } finally {
      // Hide loading state
      setIsLoading(false);
    }
  };

  return (
    <div style={{ fontFamily: "Arial, sans-serif", maxWidth: "600px", margin: "0 auto" }}>
      {/* Chat history display */}
      <div 
        style={{ 
          height: "400px", 
          overflowY: "auto", 
          border: "1px solid #ccc", 
          padding: "10px",
          marginBottom: "10px",
          borderRadius: "5px",
          backgroundColor: "#f9f9f9"
        }}
      >
        {log.length === 0 ? (
          <div style={{ color: "#666", textAlign: "center", marginTop: "180px" }}>
            Type a message below to start chatting!
          </div>
        ) : (
          log.map((entry, index) => (
            <div 
              key={index} 
              style={{
                margin: "8px 0",
                padding: "8px",
                borderRadius: "5px",
                backgroundColor: entry.startsWith("🧑") ? "#e1f5fe" : 
                                 entry.startsWith("🤖") ? "#f0f4c3" :
                                 entry.startsWith("🧩") ? "#e8eaf6" : "#ffebee"
              }}
            >
              {entry}
            </div>
          ))
        )}
      </div>
      
      {/* Message input area */}
      <div style={{ display: "flex", gap: "10px" }}>
        <input
          value={msg}
          onChange={(e) => setMsg(e.target.value)}
          onKeyDown={(e) => {
            if (e.key === "Enter" && !isLoading) {
              handleSendMessage();
            }
          }}
          placeholder="Type a message and press Enter"
          style={{ 
            flex: 1, 
            padding: "10px",
            borderRadius: "5px",
            border: "1px solid #ccc"
          }}
          disabled={isLoading}
        />
        <button 
          onClick={handleSendMessage}
          disabled={isLoading || !msg.trim()}
          style={{
            padding: "10px 15px",
            backgroundColor: isLoading ? "#cccccc" : "#4caf50",
            color: "white",
            border: "none",
            borderRadius: "5px",
            cursor: isLoading ? "not-allowed" : "pointer"
          }}
        >
          {isLoading ? "Sending..." : "Send"}
        </button>
      </div>
    </div>
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

1. Type a question like "What are three ways to improve my programming skills?"
2. Press Enter or click "Send"
3. Watch as the chatbot displays:
   - Your question (in blue)
   - The AI's answer (in light green)
   - How the AI thought about your question (in purple)

![Example conversation with the chatbot](https://i.imgur.com/placeholder3.jpg)

## Common Errors and How to Fix Them

### "Cannot read properties of undefined (reading '0')"
This error happens when the API response doesn't have the structure we expect. Our updated code now handles this with error checking!

### "API key invalid" 
- Double-check your OpenAI API key in the `.env` file
- Make sure it starts with "sk-"
- Make sure there are no spaces or quotation marks around it

### "CORS error" in the console
This is a security feature of browsers. To fix it:
- Make sure you're using the latest API endpoint
- If it persists, you might need to create a simple backend server

## Make Your Chatbot Even Better!

### Show More Detailed AI Thinking
To see more detailed reasoning, find this line:
```jsx
reasoning_summary: "concise",
```

And change it to:
```jsx
reasoning_summary: "detailed",
```

### Make Your Bot Remember the Conversation
To make your bot remember previous messages, you'll need to store the conversation history. Find:

```jsx
messages: [{ role: "user", content: userMessage }],
```

And replace it with:
```jsx
messages: [...previousMessages, { role: "user", content: userMessage }],
```

Where `previousMessages` is an array of past messages you maintain in your component's state.

## Learn More
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [React Documentation](https://react.dev/learn)
- [Vite Documentation](https://vitejs.dev/guide/)
- [Complete React Chatbot Tutorial](https://www.youtube.com/results?search_query=react+openai+chatbot+tutorial)
