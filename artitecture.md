# 🧠 Complete Architecture & Code Deep Dive

Welcome to the ultimate technical guide for **ScanSense AI**. This document is designed to be a complete front-to-back textbook. 

## 🌟 What is ScanSense AI?
**ScanSense AI** is a highly specialized, multimodal medical assistant designed for both patients and healthcare professionals. Unlike generic AI chatbots, ScanSense is tightly restricted to the medical domain and will automatically reject unrelated questions (e.g., coding, cooking, or general trivia).

### 🎯 Core Characteristics
- **Multimodal Capabilities:** It doesn't just read text! You can upload full medical PDF reports or even diagnostic images (like X-Rays, MRIs, and CT Scans), and the AI will analyze them visually.
- **Strict Medical Guardrails:** A two-step validation process securely checks every uploaded file to ensure it is actually medical data before passing it to the main AI.
- **Persistent Local Memory:** Completely private local storage. Every chat session is saved offline in an SQLite database (`chat.db`), allowing you to rename, retrieve, or delete your chat histories at any time.
- **Modular & Scalable:** The codebase is heavily modularized into clean folders (`components`, `utils`, `config`) so that adding new features in the future is incredibly easy for any developer.

### 🛠️ How is it Built?
The project sits on a cutting-edge, yet incredibly beginner-friendly tech stack:
- **Frontend Framework:** **Streamlit** (Python). It allows us to build beautiful, responsive web apps using pure Python without writing raw HTML or React.
- **AI Brain:** **Llama 4 Scout (17b-16e-instruct)** via the ultra-fast **Groq** API.
- **Database:** **SQLite**. A lightweight, file-based database that requires no server setups.
- **File Processing:** **PyPDF2** for extracting text from documents, and **Pillow (PIL)** for converting images into Base64 format for the AI's "vision".

---

First, we will look at the big picture (the Folder Structure). 
Then, we will dive into **every single Python file, line by line**, pasting the exact code block and immediately explaining what it does in plain, beginner-friendly English!

---

## 📂 1. The Big Picture: Folder Mappings

The codebase is split into 5 distinct areas. Each area handles one single responsibility:
- `config/`: Hardcoded text, AI prompts, and environment variables.
- `database/`: Saving and loading from the SQLite `.db` file.
- `utils/`: Reusable background tools (reading PDFs, talking to the AI).
- `components/`: The visual building blocks you actually see on screen (Buttons, text boxes).
- `app.py`: The glue that holds the 4 folders together.

---

## 💻 2. Full Code Walkthrough (File by File)

### ⚙️ File 1: `config/settings.py` (The Settings File)
This file is the "brain setting" of the app. It holds constants.

```python
import os
from dotenv import load_dotenv

# Load environment variables
load_dotenv()
```
**Explanation:** 
- `os` allows Python to talk to your computer's operating system. 
- `dotenv` reads that hidden `.env` file you created and temporarily saves the secret `GROQ_API_KEY` into your computer's memory. This is the safest way to hide passwords instead of typing them directly into the code.

```python
# App Configuration
PAGE_TITLE = "ScanSense AI"
LAYOUT = "wide"
INITIAL_SIDEBAR_STATE = "collapsed"

# Model configuration
MODEL_NAME = "meta-llama/llama-4-scout-17b-16e-instruct"
GROQ_API_KEY = os.getenv("GROQ_API_KEY")
```
**Explanation:** 
- `PAGE_TITLE` is what shows up on your web browser's tab at the very top of your screen. 
- `MODEL_NAME` tells the AI server exactly which "brain" to spin up. We are using **Llama 4 Scout**, a fast and highly capable multimodal LLM.
- `os.getenv` grabs that secret API key from memory so we can use it later.

```python
SYSTEM_PROMPT = """You are a helpful and professional medical assistant. 
Your role is to assist with medical analysis, health inquiries, and understanding medical documents or images.
You must STRICTLY REFUSE to answer any questions that are not related to the medical or health field (e.g., recipes, coding, general trivia, entertainment, etc.).
If a user asks a non-medical question, politely decline and state that you are a specialized medical AI analyzer designed to help with health-related queries only.
"""
```
**Explanation:** 
- `SYSTEM_PROMPT` is the most important "hidden" text in the app. Whenever you start a chat, this text is secretly sent to the AI behind the scenes. It forces the AI into character (a medical professional) and sets strict guardrails prohibiting it from answering non-medical queries.

---

### 💾 File 2: `database/db_manager.py` (The Save System)
This file handles the SQLite database, effectively serving as the permanent memory for your chats so you don't lose them when you refresh the page.

```python
import sqlite3
import datetime

DB_NAME = "chat.db"
```
**Explanation:** 
- We import `sqlite3`, which is Python's built-in, lightweight database manager. No massive SQL servers required! 
- `DB_NAME` tells Python to save everything into a simple file called `chat.db`.

```python
def init_db():
    conn = sqlite3.connect(DB_NAME)
    c = conn.cursor()
    c.execute('''
        CREATE TABLE IF NOT EXISTS sessions (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )
    ''')
    # ... (skipping second create table for brevity)
    conn.commit()
    conn.close()
```
**Explanation:** 
- The `init_db()` function runs when your app starts. 
- It opens a connection to `chat.db` (creating the file if it doesn't exist). 
- `CREATE TABLE IF NOT EXISTS`: A safety check that only creates tables the very first time you run the app. 
- The `sessions` table tracks individual chats (like folders). It assigns a unique ID (`AUTOINCREMENT`) to every new chat.
- We `commit()` (save) our changes and `close()` the connection to free up the computer's memory.

```python
def create_session(title=None):
    if not title:
        title = f"Chat {datetime.datetime.now().strftime('%Y-%m-%d %H:%M')}"
    conn = sqlite3.connect(DB_NAME)
    c = conn.cursor()
    c.execute("INSERT INTO sessions (title) VALUES (?)", (title,))
    session_id = c.lastrowid
    conn.commit()
    conn.close()
    return session_id
```
**Explanation:** 
- `create_session` prepares a new chat room. If no title is given, it auto-generates a name like "Chat 2026-03-18 10:30" using Python's `datetime` module.
- It runs an `INSERT INTO` SQL command to write that title into the database. 
- `lastrowid` grabs the ID number that SQLite randomly gave this new chat, and returns it so our app knows which chat we are currently inside!

```python
def get_chat_history(session_id):
    conn = sqlite3.connect(DB_NAME)
    conn.row_factory = sqlite3.Row
    c = conn.cursor()
    c.execute("SELECT role, content, image FROM messages WHERE session_id = ? ORDER BY created_at ASC", (session_id,))
    rows = c.fetchall()
    conn.close()
    
    messages = []
    for row in rows:
        msg = {"role": row["role"], "content": row["content"]}
        if row["image"]: msg["image"] = row["image"]
        messages.append(msg)
    return messages
```
**Explanation:** 
- When you click a past chat on the sidebar, `get_chat_history()` loads it!
- `row_factory = sqlite3.Row` is a magic trick that lets us access SQL data like a dictionary (`row["content"]`) instead of confusing lists.
- `SELECT ... WHERE session_id = ?` asks the database to ONLY give us messages belonging to the chat we just clicked.
- `ORDER BY created_at ASC` guarantees the messages are loaded chronologically from oldest (top) to newest (bottom).
- We loop through the SQL rows, package them into a python list `[]`, and hand them to Streamlit to draw on screen.

---

### 📂 File 3: `utils/file_handlers.py` (The Document Reader)
This file handles "vision" and "reading" by dissecting files you upload so the AI can understand them.

```python
import PyPDF2
import base64
import io
from PIL import Image

def extract_text_from_pdf(pdf_file):
    reader = PyPDF2.PdfReader(pdf_file)
    text = ""
    for page in reader.pages:
        text += page.extract_text()
    return text
```
**Explanation:** 
- When you upload a `.pdf`, you can't just send the raw file to an AI. It needs text!
- `PyPDF2` opens the PDF file securely. 
- We create an empty string: `text = ""`. 
- The `for` loop cycles through every single page in the document, extracts the human-readable text, glues it to our giant `text` string, and hands that string back to the app.

```python
def encode_image(image):
    buffered = io.BytesIO()
    image.convert('RGB').save(buffered, format="JPEG")
    return base64.b64encode(buffered.getvalue()).decode('utf-8')
```
**Explanation:** 
- AI models on the internet accept images in a mathematical text format called **Base64**.
- `encode_image()` takes the picture you uploaded using `PIL` (Python Imaging Library).
- `convert('RGB')` strips away transparency (like PNG backgrounds) to ensure perfect compatibility.
- `io.BytesIO()` is a virtual folder in your RAM. We save the image there temporarily, convert all its pixels into raw binary code, then encode it to `base64` so the Groq API can safely swallow it over an HTTP network request.

---

### 🧠 File 4: `utils/ai_assistant.py` (The Logic & Brains)
This connects your app straight to Groq's supercomputers and implements smart security.

```python
from groq import Groq
import streamlit as st
from config.settings import GROQ_API_KEY, MODEL_NAME

def get_groq_client():
    if not GROQ_API_KEY:
        st.error("GROQ_API_KEY not found in environment variables.")
        st.stop()
    return Groq(api_key=GROQ_API_KEY)
```
**Explanation:** 
- Without an API key, we have no brain. If the key is missing from `.env`, `st.error` paints a red warning box on the screen, and `st.stop()` instantly kills the application to prevent further crash errors!
- Otherwise, we return a `Groq` client object—our VIP pass to talk to the AI.

```python
def is_medical_content(content, content_type):
    client = get_groq_client()
    
    if content_type == 'image':
        messages = [{
            "role": "user",
            "content": [
                {"type": "text", "text": "Is this image related to the medical field? Strict YES or NO."},
                {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{content}"}}
            ]
        }]
    # ... handles PDF chunk similarly by taking text[:2000] (first 2000 characters to save time)
    
    try:
        completion = client.chat.completions.create(
            messages=messages, model=MODEL_NAME, temperature=0, max_tokens=10
        )
        response = completion.choices[0].message.content.strip().upper()
        return "YES" in response
```
**Explanation:** 
- **The Bouncer Check!** Before processing your file in the main chat, this function opens a *secret, invisible* background conversation with the AI.
- It asks a simple YES/NO question about your file. 
- Look at `temperature=0`! AI usually has a "temperature" (creativity). By setting it to `0`, we force the AI to be extremely robotic, unimaginative, and precise.
- Look at `max_tokens=10`. We only want a YES or NO. Setting tokens to 10 ensures the AI doesn't waste time typing a long paragraph, keeping the response blazing fast!
- If the AI replies with "YES", the file is approved.

---

### 🎨 File 5: `components/sidebar.py` (The Sidebar Layout)
This file renders the entire dark column on the left containing history and settings.

```python
import streamlit as st
from database.db_manager import get_all_sessions, get_chat_history, update_session_title, delete_session

def render_sidebar():
    def reset_chat():
        st.session_state.chat_history = []
        st.session_state.current_session_id = None
        # ... clears context and images
```
**Explanation:** 
- `render_sidebar()` is the master drawing function for the left panel.
- Inside it, `reset_chat()` is a helper function. Think of your `st.session_state` as a giant whiteboard. When you click "New Chat", this function wipes the whiteboard clean! It erases `chat_history` and turns your `current_session_id` to `None`. 

```python
    with st.sidebar:
        st.title("History")
        st.button("New Chat", on_click=reset_chat, use_container_width=True)
        st.divider()
        sessions = get_all_sessions()
```
**Explanation:** 
- `with st.sidebar:` is Streamlit's way of saying "draw everything underneath this line on the left-hand panel".
- `st.button` draws a button that fills the entire width `use_container_width=True`. When clicked, it automatically runs our `reset_chat` wipe function!
- `sessions = get_all_sessions()` asks standard SQlite to hand us all past chats saved locally.

```python
        for session in sessions:
            col1, col2 = st.columns([0.85, 0.15])
            with col1:
                if st.button(session["title"], key=f"session_{session['id']}"):
                    st.session_state.current_session_id = session["id"]
                    st.session_state.chat_history = get_chat_history(session["id"])
                    st.rerun()
```
**Explanation:** 
- A `for` loop cycles through every saved chat.
- We artificially split the sidebar space into two columns: `85%` for the big title button, `15%` for the three-dots menu.
- If you click the title button (`col1`), it immediately updates the `session_state` whiteboard with that old ID, downloads its history from the DB using `get_chat_history`, and calls `st.rerun()`.
- `st.rerun()` literally restarts the python script block instantly from the top so the page updates with your past chat!

```python
            with col2:
                with st.popover("⋮"):
                    new_title = st.text_input("Rename", value=session["title"], key=f"rename_{session['id']}")
                    if st.button("Save", key=f"save_{session['id']}"):
                        update_session_title(session["id"], new_title)
                        st.rerun()
                    if st.button("🗑️ Delete", key=f"delete_{session['id']}"):
                        delete_session(session["id"])
                        st.rerun()
```
**Explanation:** 
- In the tiny `15%` column (`col2`), we build `st.popover("⋮")`. A popover is a hidden menu that reveals elements only when clicked.
- Inside the popover, we add a text box (`st.text_input`) and two buttons (`Save` and `Delete`). All these elements are carefully assigned a unique `key` combined with the `session['id']` so Streamlit never mixes up which chat you are interacting with.
- If deleted, it runs our SQL deletion handler, and forces an immediate page `st.rerun()` to make the chat physically vanish from the menu.

---

### 💬 File 6: `components/chat_interface.py` (The Screen Engine)
The largest UI module responsible for the central chat screen.

```python
def render_chat_interface():
    messages_container = st.container()

    with messages_container:
        for message in st.session_state.chat_history:
            role = message["role"]
            avatar = "🤖" if role == "assistant" else "👤"
            with st.chat_message(role, avatar=avatar):
                st.markdown(message["content"])
```
**Explanation:** 
- `st.container()` restricts all our chat bubbles into a central invisible box on your main screen.
- The `for message in chat_history:` loop is standard logic. It checks if the message came from the user or the robot, gives it an emoji avatar corresponding to the role, and uses `st.chat_message` to draw that beautiful padded chat bubble! `st.markdown()` renders the bold/italic text properly.

```python
    prompt = st.chat_input("Ask something...", key="chat_input", accept_file=True, file_type=["pdf", "png", "jpg", "jpeg"])

    if prompt:
        user_text = prompt.text if hasattr(prompt, "text") else getattr(prompt, "text", prompt)
        uploaded_file = prompt.files[0] if (hasattr(prompt, "files") and prompt.files) else None
```
**Explanation:** 
- `st.chat_input` is Streamlit's hero widget. Because we pass `accept_file=True`, it places a sleek paperclip icon inside the input bar instead of needing an ugly separate upload button!
- When you hit enter, ALL the code below `if prompt:` executes. 
- Because `prompt` can now contain BOTH text AND a file (a dictionary), we have multiple checks like `hasattr` to safely extract exactly what you typed into `user_text` and exactly what image you attached into `uploaded_file`. 

**(File Validation & Processing Omitted for Brevity - It follows similar PyPDF2/Image.open logic explained earlier.)**

```python
        if user_text:
            if st.session_state.current_session_id is None:
                st.session_state.current_session_id = create_session()
```
**Explanation:** 
- Notice the `is None` check! We ONLY create a blank folder profile inside SQLite at the EXTREME LAST SECOND. This is called "lazy initialization." If we did this earlier, every time you clicked "New Chat", an empty broken chat would save to the database unnecessarily!

```python
            st.session_state.chat_history.append(user_msg)
            save_message(st.session_state.current_session_id, "user", user_text, encoded_image)
            
            if len(st.session_state.chat_history) == 1:
                new_title = str(user_text)[:30] + "..." if len(str(user_text)) > 30 else str(user_text)
                try:
                    update_session_title(st.session_state.current_session_id, new_title)
                except Exception:
                    pass
```
**Explanation:** 
- Here, we save your prompt into the whiteboard `append(user_msg)`, and permanently onto the hard drive `save_message`.
- **Auto Naming**: Then, if `len == 1` (meaning this is the absolute FIRST message of the session), it slices the very first `[:30]` thirty characters of your prompt to invent a name for the chat and immediately updates it in SQLite! Extremely elegant logic heavily used in ChatGPT.

```python
            client = get_groq_client()
            with messages_container:
                with st.chat_message("assistant", avatar="🤖"):
                    placeholder = st.empty()
                    full_response = ""
                    # ... [Appends history array including SYSTEM_PROMPT to memory block named `messages`] 

                    completion = client.chat.completions.create(messages=messages, model=MODEL_NAME, stream=True, temperature=0.7)
                    for chunk in completion:
                        if chunk.choices[0].delta.content:
                            full_response += chunk.choices[0].delta.content
                            placeholder.markdown(full_response + "▌")
                    placeholder.markdown(full_response)
```
**Explanation:** 
- Now it's the AI's turn! The `stream=True` argument inside `completions.create` is what gives the app that professional "typing speed" animation effect.
- We create an empty box on screen: `placeholder = st.empty()`. 
- As Groq shoots tiny packets of letters (called "chunks") over the network, our `for chunk in completion` loop catches them. It continuously adds the new letter to `full_response` and redraws `placeholder.markdown()` over and over.
- We add a playful `▌` cursor at the end to make it look even more authentic while it types!

---

### 🧩 File 7: `app.py` (The Entry Point Frame)
This brings the 4 folders completely together in just ~60 lines.

```python
import streamlit as st
from config.settings import PAGE_TITLE, LAYOUT, INITIAL_SIDEBAR_STATE
from database.db_manager import init_db
from components.sidebar import render_sidebar
from components.chat_interface import render_chat_interface

def main():
    # Initialize SQLite database schema
    init_db()

    # Apply global styles ...
    st.markdown('''
        <style>
        [data-testid="stHeader"]::after { content: "ScanSense AI 🧿"; ... }
        div[data-testid="stPopover"] button p + * { display: none !important; }
        </style>
        ''', unsafe_allow_html=True)
    
    initialize_session_state()

    # Render App Components
    render_sidebar()
    render_chat_interface()

if __name__ == "__main__":
    main()
```
**Explanation:** 
- In Python, `if __name__ == "__main__":` ensures this code only automatically plays if you specifically run `streamlit run app.py` on this file! 
- The `<style>` chunk uses advanced CSS (Cascading Style Sheets) to inject raw styling instructions onto the page. For example, `p + * { display: none !important; }` aggressively hunts down and wipes out the ugly default dropdown arrows on the popovers!
- The app fundamentally just calls `init_db()` (builds tables), `initialize_session_state()` (wipes the whiteboard), and then draws the two big puzzle pieces `render_sidebar()` and `render_chat_interface()`.

You have now mastered the **ScanSense AI** backend flow! No matter what features you build in the future, you have the absolute blueprint inside this document.
