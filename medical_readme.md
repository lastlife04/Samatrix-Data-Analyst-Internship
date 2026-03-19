# ScanSense AI 🧿

Welcome to **ScanSense AI**! This is a specialized, AI-powered medical assistant. It helps users analyze medical reports (PDFs) and medical images (X-rays, MRIs, etc.) using the powerful Llama 4 Scout model.

This document will guide you step-by-step on how to run this project on your own computer, even if you are a complete beginner!

If you want to understand how the code works, please read the [ARCHITECTURE.md](ARCHITECTURE.md) file!

---

## 🚀 How to Run the App (Step-by-Step)

### Step 1: Install Python
You need to have Python installed on your computer. 
- Go to [python.org](https://www.python.org/downloads/) and download the latest version for your operating system (Windows, Mac, or Linux).
- **Important for Windows users:** When installing, make sure to check the box that says **"Add Python to PATH"**.

### Step 2: Get a Free AI API Key
This app uses an AI brain from a company called Groq. You need a free key to connect to it.
1. Go to [Groq Console](https://console.groq.com/keys).
2. Create a free account.
3. Click on "Create API Key" and copy the long string of text. Keep it secret!

### Step 3: Set Up the Project
1. Open up your terminal (or Command Prompt / PowerShell on Windows) and navigate to the folder where this project is saved.
2. Inside this folder, create a new file named exactly `.env` (don't forget the dot at the beginning).
3. Open the `.env` file in notepad or any text editor and paste your API key like this:
   ```env
   GROQ_API_KEY=your_copied_api_key_here
   ```
4. Save the file.

### Step 4: Install the Required Packages
Your computer needs to download some open-source libraries (like Streamlit) to run the code.
Run this command in your terminal:
```bash
pip install -r requirements.txt
```
*(Wait a minute or two for everything to download and install).*

### Step 5: Start the App!
Finally, you are ready to launch the app. Run this command in your terminal:
```bash
python -m streamlit run app.py
```
A browser window will automatically pop up at `http://localhost:8501`, and you can start chatting with your AI!

---

## ✨ Features
- **Strictly Medical:** The AI validates every file you upload and refuses to answer non-medical questions.
- **Image & PDF Support:** Upload your medical PDFs or X-rays directly into the chat!
- **History Saving:** Every chat you have is automatically saved inside the `chat.db` file so you never lose your history. You can rename or delete them from the sidebar.
