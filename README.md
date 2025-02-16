# Day 5 AI Autocorrection and Playground
---
## Features in the Build
- Auto-Completion: Predicts the next word/token while typing.  
- Auto-Correction: Fixes syntax errors (e.g., missing semicolons, misspelled keywords).  
- FastAPI Backend: Provides AI-driven corrections and predictions.  
- rontend Editor: Implements real-time suggestions in Darion Playground.  

---

## **1. Backend - FastAPI for AI Code Suggestions**
This API will:
- Accept **partial Darion code** as input.
- Use **AI models** (RapidFuzz for fuzzy matching) for **auto-completion & syntax correction**.
- Return **corrected code & suggestions**.

Let's create the **FastAPI backend**:  

### *Backend Code (FastAPI)*
This script will:
- Provide **auto-completion** using **predefined syntax rules**.
- Use **fuzzy matching** to detect and fix syntax errors.  

```python
from fastapi import FastAPI
from pydantic import BaseModel
from rapidfuzz import process

app = FastAPI()

# Predefined keywords for auto-completion
darion_keywords = ["let", "print", "function", "for each", "if", "else", "return", "import", "while", "do"]

# Sample syntax corrections
syntax_corrections = {
    "funtion": "function",
    "prnt": "print",
    "retun": "return",
    "fr each": "for each",
    "wihle": "while"
}

class CodeRequest(BaseModel):
    code: str

@app.post("/autocomplete/")
async def autocomplete(request: CodeRequest):
    """Returns suggested words for auto-completion."""
    input_text = request.code.split()[-1]  # Last word
    suggestions = process.extract(input_text, darion_keywords, limit=3)
    return {"suggestions": [s[0] for s in suggestions]}

@app.post("/correct/")
async def correct_code(request: CodeRequest):
    """Returns corrected syntax if any mistakes are found."""
    words = request.code.split()
    corrected_words = [syntax_corrections.get(word, word) for word in words]
    return {"corrected_code": " ".join(corrected_words)}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### *Create/Edit `tailwind.config.js`*
If you don’t have a `tailwind.config.js`, generate it by running:  
```js
npx tailwindcss init
```
Then, replace the contents with:  

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./src/**/*.{js,jsx,ts,tsx}", // Ensure Tailwind scans your React files
    "./public/index.html",
  ],
  theme: {
    extend: {
      colors: {
        darionBlue: "#1E40AF", // Custom color example
      },
    },
  },
  plugins: [],
};
```

---

### *Ensure Tailwind is Included in Your CSS*
In `index.css` or `App.css`, add:  
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

### *Restart Your Dev Server*
Changes in `tailwind.config.js` require a restart:  
```
npm run dev  # or npm start
```

---
## **2. Frontend - Darion Playground Editor with AI Suggestions**
### 1. **Set Up a React Environment**:

If you don't already have a React app set up, you can create one using `create-react-app` (or any React setup you prefer). Here's how you can get started:

#### Using Create React App:
1. **Install Node.js** (if not installed yet):  
   You can download and install Node.js from [here](https://nodejs.org/).

2. **Create a new React app**:
   Open a terminal and run the following command to create a new React application:
   ```bash
   npx create-react-app darion-playground
   ```

3. **Navigate to your project directory**:
   ```bash
   cd darion-playground
   ```

4. **Install dependencies (if necessary)**:
   If you're using `fetch` (which is built into modern browsers), you don't need to install anything special. However, if you're working with older environments or want to ensure compatibility, you can install `whatwg-fetch`:
   ```bash
   npm install whatwg-fetch
   ```

5. **Start the development server**:
   ```bash
   npm start
   ```
   This will launch your React app in the browser (usually at `http://localhost:3000`).

### 2. **Add the Code to Your React App**:
   You need to place the `Playground` component in your app's file structure.

#### Steps:
1. **Navigate to the `src` folder**:
   In your React app, go to the `src` folder. This is where you will put your React components.

2. **Create a new file for the component**:
   Create a new file called `Playground.js` inside the `src` folder and paste the code you provided (the enhanced version with loading/error handling) into that file.

3. **Import and use the `Playground` component**:
   In the `src/App.js` file, import the `Playground` component you just created and render it inside your `App` component. For example:

```javascript
// src/App.js
import React from 'react';
import Playground from './Playground';  // Import the Playground component

function App() {
  return (
    <div className="App">
      <Playground />  {/* Use the Playground component */}
    </div>
  );
}

export default App;
```


 
 
### 3. **Ensure the API is Running**:
For this code to work, you need the backend API to be running locally at `http://localhost:8000`. Make sure that:
- You have the server running on `http://localhost:8000` with the relevant endpoints (`/autocomplete/` and `/correct/`).
- The backend should handle POST requests at those endpoints and return the appropriate suggestions and corrected code.

If you're using a framework like Django, Flask, or Express.js for the backend, make sure the server is running and is able to handle the requests.

### 4. **Styling (Optional)**:
If you want to make sure that TailwindCSS is applied (since you're using Tailwind in your code), ensure it's set up in your project. You can follow the TailwindCSS installation guide [here](https://tailwindcss.com/docs/installation).

Alternatively, if you don't want to use TailwindCSS, you can replace the class names with regular CSS styles or any other CSS framework.

### 5. **Run the App**:
Once everything is set up, go ahead and run your app with `npm start`:

```bash
npm start
```


---

The frontend will:
- Call the **FastAPI backend** for **auto-completion & corrections**.
- Display **real-time suggestions** while typing.  

### ** Frontend Code (React + Tailwind + Fetch API)**
This React-based code will:
- Capture **code input**.
- Request **auto-completion & corrections** from the FastAPI backend.
- Display **inline suggestions** in the editor.

```javascript
import { useState, useEffect } from "react";

export default function Playground() {
  const [code, setCode] = useState("");
  const [suggestions, setSuggestions] = useState([]);
  const [correctedCode, setCorrectedCode] = useState("");

  useEffect(() => {
    if (code.length > 1) {
      fetchSuggestions();
    }
  }, [code]);

  const fetchSuggestions = async () => {
    const response = await fetch("http://localhost:8000/autocomplete/", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ code }),
    });
    const data = await response.json();
    setSuggestions(data.suggestions);
  };

  const fetchCorrection = async () => {
    const response = await fetch("http://localhost:8000/correct/", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ code }),
    });
    const data = await response.json();
    setCorrectedCode(data.corrected_code);
  };

  return (
    <div className="p-4 bg-gray-900 text-white h-screen">
      <h1 className="text-xl mb-4">Darion Playground</h1>
      <textarea
        className="w-full p-2 bg-gray-800 text-white rounded-lg"
        rows="6"
        value={code}
        onChange={(e) => setCode(e.target.value)}
        placeholder="Start typing Darion code..."
      />
      <div className="mt-2">
        {suggestions.length > 0 && (
          <div className="p-2 bg-gray-700 rounded-md">
            <strong>Suggestions:</strong> {suggestions.join(", ")}
          </div>
        )}
      </div>
      <button
        onClick={fetchCorrection}
        className="mt-4 bg-blue-500 p-2 rounded-md"
      >
        Correct Code
      </button>
      {correctedCode && (
        <div className="mt-2 p-2 bg-green-700 rounded-md">
          <strong>Corrected Code:</strong> {correctedCode}
        </div>
      )}
    </div>
  );
}
```

---

## **3️. Deployment & Integration**
### ** Steps to Run**
1. **Run Backend** (FastAPI)  
   ```bash
   pip install fastapi uvicorn rapidfuzz pydantic
   python backend.py
   ```

2. **Run Frontend** (React)  
   ```bash
   npm install
   npm start
   ```

---

## ** Final Features in Darion Playground**
**Real-time Auto-Completion** while typing.  
**AI-powered Syntax Correction** for errors.  
**FastAPI Backend** for AI suggestions.  
**React-based Playground UI** for smooth experience.  

