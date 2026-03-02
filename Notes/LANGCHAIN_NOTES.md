# 🔗 LangChain Learning Notes - Complete Beginner Guide

---

## 📚 What is LangChain?

Think of **LangChain** as a toolkit that makes it easy to build applications with AI language models. Instead of dealing with complex APIs directly, LangChain provides simple building blocks you can connect together like LEGO pieces to create powerful AI applications.

---

## 📁 Folder 1: 01- Basics (Using Different AI Models)

This folder teaches you how to connect to different AI models and ask them questions.

### 📄 1. **chatmodels.py** - Using OpenRouter with Gemma Model

**What it does:**
- Connects to **OpenRouter** (an API service) to use the **Gemma 3 model**
- Asks the AI: "Capital of India"
- The AI responds with the answer

**How it works (Step by Step):**
1. **Import libraries**: Load the tools we need
2. **Load API key**: Get the API key from your `.env` file (a secret file with passwords)
3. **Create model**: Set up ChatOpenAI to use Gemma 3
4. **Ask question**: Use `llm.invoke()` to send a message to the AI
5. **Get response**: Print the AI's answer

**Simple Analogy:**
It's like calling a customer service bot:
- `Load API key` = Getting permission to call
- `Create model` = Selecting which bot to call
- `Invoke` = Asking a question
- Response = Getting the answer back

```python
# The model created with these settings:
- base_url: "https://openrouter.ai/api/v1"  # Where to send the request
- api_key: Your secret key  # Proof that you're allowed to use it
- model: "google/gemma-3-12b-it:free"  # Which AI model to use (Gemma)
```

---

### 📄 2. **GoogleGeminiModel.py** - Using Google's Gemini Model

**What it does:**
- Uses **Google's Gemini 1.5 Flash** model directly
- Asks: "What is the capital of India?"
- Gets a response from Gemini

**Key Differences from chatmodels.py:**
- Uses `ChatGoogleGenerativeAI` instead of `ChatOpenAI`
- Connects directly to Google's API (not through OpenRouter)
- Simpler setup - just needs the model name

**Real-world use:**
If you want to use Google's advanced AI model (Gemini), this is how you do it.

---

### 📄 3. **HugginFace.py** - Using HuggingFace's TinyLlama Model

**What it does:**
- Connects to a **lightweight AI model** called "TinyLlama"
- This model is smaller and faster (good for older computers)
- Asks the same question: "What is the capital of India?"

**Why this is cool:**
- Uses `HuggingFaceEndpoint` - connects to HuggingFace's servers
- Then wraps it with `ChatHuggingFace` to make it easier to use
- `max_new_tokens=256` - limits the response length to 256 words
- `temperature=0.7` - controls creativity (0=boring, 1=creative)

**Simple Comparison:**
| Model | Company | Speed | Quality |
|-------|---------|-------|---------|
| Gemma 3 | Google | Fast | High |
| Gemini | Google | Medium | Very High |
| TinyLlama | HuggingFace | Very Fast | Medium |

---

## 📁 Folder 2: 02- Prompting (Creating Interactive User Input)

### 📄 **prompt_ui.py** - Web Interface with Streamlit

**What it does:**
- Creates a **web interface** (like a website) where users can type questions
- User types a prompt → AI answers → Shows on website

**How it works:**
1. Streamlit creates a header: "Research Tool"
2. Creates a text box for user input
3. User clicks "Submit" button
4. Code sends the question to Gemma model
5. AI responds and shows the answer on the web page

**Simple Breakdown:**
```
User Interface (Website)
    ↓
User types: "What is AI?"
    ↓
Question sent to Gemma
    ↓
AI generates answer
    ↓
Answer displayed on website
```

**Why it's useful:**
- No need to run Python directly
- Users can interact through a browser
- Perfect for creating chatbots or research tools

---

## 📁 Folder 3: 03- EmbeddedModels (Converting Text to Numbers)

This is the **most important** concept! Learn what embeddings are.

### 📄 **Understanding Embeddings First:**

**What are Embeddings?**
Embeddings convert text into **numbers** that computers can understand and compare.

**Simple Analogy:**
Imagine you have books in different languages:
- English: "The cat sat on the mat"
- Spanish: "El gato se sentó en la estera"
- They mean the same thing!

Embeddings convert BOTH to the same "number pattern" so computers know they mean the same thing.

```
Text: "Apple"
↓
Embedding: [0.25, -0.45, 0.89, 0.12, ...] (384 numbers)

Text: "Fruit"
↓
Embedding: [0.28, -0.42, 0.87, 0.15, ...] (384 numbers)

Result: These two embeddings are similar, so computer knows "Apple" is similar to "Fruit"
```

---

### 📄 1. **check_dimensions.py** - Testing Embedding Size Consistency

**What it does:**
- Tests if **short text** and **long text** produce embeddings of the same size
- Shows that all text converts to the same number of dimensions

**Key Finding:**
```
Short text: "Hi" → 384 numbers
Long text: "Delhi is the capital..." (10 repetitions) → 384 numbers

Are dimensions equal? YES!
```

**Why this matters:**
- All text becomes the same size vector (384 numbers)
- This makes it easy to compare different texts
- Computer can calculate how similar two texts are

---

### 📄 2. **embedding_hf_local.py** - Local HuggingFace Embeddings

**What it does:**
- Downloads an embedding model locally (on your computer)
- Converts the text: "Delhi is the capital of India." into numbers

**Simple Steps:**
1. Load the model: `sentence-transformers/all-MiniLM-L6-v2`
   - This is a free, lightweight model
   - No internet needed after download
2. Convert text to vector (384 numbers)
3. Print the vector length and first 5 numbers

**Why "local"?**
- Downloads once, runs offline
- Private (no data sent to servers)
- Faster (no internet delay)

**Real Output Example:**
```
Vector Length: 384
First 5 elements: [0.123, -0.456, 0.789, -0.234, 0.567]
```

---

### 📄 3. **embedding_openai_docs.py** - OpenAI Embeddings for Multiple Documents

**What it does:**
- Uses **OpenAI's embedding model** (paid, but very good)
- Converts **multiple documents** into vectors

**Documents Embedded:**
```
1. "Delhi is the capital of india"
2. "Kolkata is the capital of west Bengal"
3. "Paris is the capital of France"
```

**Process:**
1. Load OpenAI API key from `.env`
2. Create embedding model: `text-embedding-3-large`
3. Set dimension to 32 (smaller than default)
4. Use `embed_documents()` to convert all 3 texts INTO vectors
5. Print the results

**Use Case:**
- Building search engines
- Finding similar documents
- Recommendation systems

---

### 📄 4. **embedding_openai_query.py** - OpenAI Embeddings for Search Queries

**What it does:**
Similar to above, but:
- Uses `embed_documents()` but passes a **string** instead of a list
- Note: There's a typo in the code (`text-embedding-3-lager` should be `text-embedding-3-large`)

**What it tries to do:**
Convert the search query "capital of india" into an embedding vector.

**Real Use Case:**
1. User searches: "capital of india"
2. Convert search query to embedding
3. Find embeddings most similar to this query
4. Show matching documents

---

## 📁 Folder 4: 04- langchain-structured-output (Advanced Data Handling)

This folder shows **professional ways** to get structured data from AI models.

### **Background: What are TypedDict and Pydantic?**

**Pydantic:**
- A Python library for validating and managing data
- Uses class syntax
- Very strict validation
- Best for: Complex data with rules

**TypedDict:**
- Python's built-in type hint
- Uses dictionary syntax
- Lightweight validation
- Best for: Simple data structures

---

### 📄 1. **pydantic_demo.py** - Working with Pydantic Classes

**What it does:**
- Shows how Pydantic works for data validation
- Creates a Student class with rules

**The Student Class:**
```python
class Student(BaseModel):
    name: str = 'Ankush'  # Default name
    age: Optional[int] = None  # Age can be empty
    email: EmailStr  # Must be valid email
    cgpa: float = Field(gt=0, lt=10, default=5)  # Must be between 0-10
```

**What happens:**
```python
new_student = {'age': "22", 'email': 'abc@yahoo.com'}
student = Student(**new_student)  # Creates object

# Age "22" (string) is automatically converted to 22 (number)!
# This is called coercion
```

**Output:**
```json
{
  "name": "Ankush",
  "age": 22,
  "email": "abc@yahoo.com",
  "cgpa": 5.0
}
```

**Key Feature:**
- Automatic type conversion
- Data validation
- Can export to JSON easily

---

### 📄 2. **typeddict_demo.py** - Simple Type Hints with TypedDict

**What it does:**
- Shows Python's lighter-weight way to define data structures
- No class, just a dictionary with type hints

**The Person TypedDict:**
```python
class Person(TypedDict):
    name: str
    age: int

# Usage:
new_person: Person = {'name': 'ankush', 'age': '22'}
print(new_person)
# Output: {'name': 'ankush', 'age': '22'}
```

**Difference from Pydantic:**
- TypedDict is purely a type hint (doesn't enforce)
- Pydantic validates and converts types
- TypedDict is more lightweight
- Pydantic is more powerful

---

### 📄 3. **with_structured_output_json.py** - Getting JSON from AI

**What it does:**
- Uses `with_structured_output()` with a JSON schema
- Analyzes a Samsung Galaxy S24 Ultra review
- Extracts: themes, summary, sentiment, pros, cons, reviewer name

**The JSON Schema:**
```json
{
  "key_themes": ["array", "of", "strings"],
  "summary": "string",
  "sentiment": "pos or neg",
  "pros": ["array", "of", "strings"],
  "cons": ["array", "of", "strings"],
  "name": "string"
}
```

**How it works:**
```
Long review text
        ↓
Model receives schema + instruction
        ↓
Model analyzes and returns JSON following the schema
        ↓
Python gets structured data (not messy text)
```

**Real Output Example:**
```json
{
  "key_themes": ["processor", "camera", "battery", "design"],
  "summary": "Excellent phone but expensive and heavy",
  "sentiment": "pos",
  "pros": ["Fast processor", "Good camera", "Long battery"],
  "cons": ["Too heavy", "Expensive", "Bloatware"],
  "name": "Ankush Choudhary"
}
```

---

### 📄 4. **with_structured_output_pydantic.py** - Using Pydantic Classes

**What it does:**
Same as JSON version, but uses **Pydantic class** instead of JSON schema

**The Review Class:**
```python
class Review(BaseModel):
    key_themes: list[str] = Field(description="...")
    summary: str = Field(description="...")
    sentiment: Literal["pos", "neg"] = Field(description="...")
    pros: Optional[list[str]] = Field(default=None, description="...")
    cons: Optional[list[str]] = Field(default=None, description="...")
    name: Optional[str] = Field(default=None, description="...")
```

**Key Features:**
- Type checking: Must be list of strings, not numbers
- Literal: Sentiment MUST be "pos" or "neg", nothing else
- Optional: Can be empty with default=None

**Usage:**
```python
structured_model = model.with_structured_output(Review)
result = structured_model.invoke(review_text)
# Result is a Review object with all validations applied!
```

**Advantage:**
- Guaranteed data types
- Validation built-in
- IDE autocomplete (when you type `result.pros`, IDE suggests it)

---

### 📄 5. **with_structured_output_typeddict.py** - Using TypedDict

**What it does:**
Same functionality, but using **TypedDict** instead of Pydantic

**The Review TypedDict:**
```python
class Review(TypedDict):
    key_themes: Annotated[list[str], "Write down all the key themes..."]
    summary: Annotated[str, "A brief summary..."]
    sentiment: Annotated[Literal["pos", "neg"], "Return sentiment..."]
    pros: Annotated[Optional[list[str]], "Write down all the pros..."]
    cons: Annotated[Optional[list[str]], "Write down all the cons..."]
    name: Annotated[Optional[str], "Write the name..."]
```

**Key Difference:**
- Uses `Annotated` to add descriptions
- Lighter weight than Pydantic
- Less validation

**When to use:**
- Pydantic: Need strict validation
- TypedDict: Just need type hints

---

### 📄 6. **with_structured_output_llama.py** - Local Model with Structured Output

**What it does:**
- Uses **local TinyLlama model** (not cloud-based)
- Gets structured output using Pydantic
- Reviews Samsung phone

**Important Difference:**
```python
llm = HuggingFaceEndpoint(
    repo_id="TinyLlama/TinyLlama-1.1B-Chat-v1.0",
    task="text-generation"
)
model = ChatHuggingFace(llm=llm)
```

- No API key needed (runs locally)
- Good for privacy
- Slower than cloud models but free to use

**Use Case:**
When you want AI without sending data to the cloud!

---

### 📄 **json_schema.json** - Example Schema File

```json
{
    "title": "student",
    "description": "schema about students",
    "type": "object",
    "properties": {
        "name": "string",
        "age": "integer"
    },
    "required": ["name"]
}
```

**What this means:**
- Title: "student" (what this is about)
- Properties: What fields it has (name, age)
- Required: "name" must be provided, age is optional
- Type: This is an object (like a dictionary)

---

### 📄 **students_dataset.csv** - Sample Data File

**What it contains:**
- A sample CSV file with student information
- Used for testing embeddings and structured output
- Contains rows of student data for practice

---

## 📁 Folder 5: 05- langchain-output-parsers (Organizing AI Responses)

The problem: AI responses are messy! When you ask for structured data, the AI might format it weirdly. **Output Parsers** clean this up.

### 📄 1. **jsonoutputparser.py** - Parsing JSON Responses

**What it does:**
- Asks AI for "5 facts about black hole"
- Tells AI to format response as **JSON** (structured data)
- Parses (cleans up) the JSON response

**JSON Format:**
```json
{
  "fact_1": "Black holes have gravity so strong...",
  "fact_2": "Nothing can escape...",
  "fact_3": "They warp space-time..."
}
```

**How it works (Chain):**
```
User asks about: "black hole"
        ↓
PromptTemplate fills in the topic
        ↓
Model generates response (as JSON)
        ↓
JsonOutputParser cleans it up
        ↓
Python gets a clean Python dictionary
```

**Code Pattern:**
```python
parser = JsonOutputParser()
template = PromptTemplate(..., partial_variables={'format_instruction': parser.get_format_instructions()})
chain = template | model | parser  # Pipe operator chains them
result = chain.invoke({'topic': 'black hole'})
```

**Key Learning:**
The `|` (pipe) operator connects components:
- Template → Model → Parser
- Like a factory line!



---

### 📄 2. **pydanticoutputparser.py** - Using Python Classes for Structure

**What it does:**
- Defines a **Python class** (Person) with specific fields
- Asks AI to generate a person with: name, age, city
- AI's response automatically becomes a Person object

**The Person Class:**
```python
class Person(BaseModel):
    name: str = "Just text"
    age: int = "Must be a number"
    city: str = "Just text"
```

**Why Pydantic?**
- Validates data (checks if it matches the class definition)
- If age = "abc", it rejects it (must be a number)
- Very powerful for real applications

**Process:**
```
"Generate a Sri Lankan person"
        ↓
AI generates: name, age, city
        ↓
Pydantic validates the data
        ↓
Returns a Person object (not just text)
```

**Advantage over JSON:**
- Type-safe (checks if types are correct)
- Can add validation (like `gt=18` means age > 18)
- Easier to work with in Python

---

### 📄 3. **stroutputparser.py** - Combining Prompts with Output Parser

**What it does:**
- Creates a **two-step chain**:
  1. Writes detailed report on "black hole"
  2. Summarizes the report to 5 lines
- Uses `StrOutputParser` to return clean string responses

**The Chain:**
```
Template 1 (Report)
    ↓
Model (generates report)
    ↓
StrOutputParser (cleans string)
    ↓
Template 2 (Summary)
    ↓
Model (summarizes)
    ↓
StrOutputParser (final clean output)
```

**Simple Analogy:**
Like an assembly line:
- Station 1: Write detailed report
- Station 2: Summarize the report
- Final product: 5-line summary

---

### 📄 4. **stroutputparser1.py** - Efficient Chaining

**What it does:**
Same as above, but **more efficient**:
```python
chain = template1 | model | parser | template2 | model | parser
result = chain.invoke({'topic': 'black hole'})
```

**Key Insight:**
- No intermediate storage
- Everything connected in one chain
- Cleaner code, better performance

---

### 📄 5. **structuredoutputparser.py** - Structured Responses with ResponseSchema

**What it does:**
- Uses `StructuredOutputParser` with `ResponseSchema`
- Asks for "3 facts about black hole"
- Forces AI to return exactly 3 facts with specific names

**The Schema:**
```python
schema = [
    ResponseSchema(name='fact_1', description='Fact 1 about the topic'),
    ResponseSchema(name='fact_2', description='Fact 2 about the topic'),
    ResponseSchema(name='fact_3', description='Fact 3 about the topic'),
]
```

**Result Format:**
```
{
    'fact_1': 'Black holes are...',
    'fact_2': 'Event horizon...',
    'fact_3': 'Time slows down...'
}
```

**When to use this vs Pydantic:**
| Feature | StructuredOutputParser | Pydantic |
|---------|------------------------|----------|
| Simple data | ✅ Good | 🔧 Overkill |
| Complex validation | ❌ No | ✅ Yes |
| Type checking | ❌ No | ✅ Yes |
| Ease of use | ✅ Easy | 🔧 Needs class |

---

## 🎯 Quick Summary & When to Use Each

### **1. Chat Models**
| Model | Source | Speed | Quality | Cost |
|-------|--------|-------|---------|------|
| Gemma 3 | OpenRouter | Fast | Good | Free |
| Gemini | Google | Medium | Excellent | Paid |
| TinyLlama | HuggingFace | V. Fast | Fair | Free |

**Choose based on:** Your budget and quality needs

---

### **2. Embeddings**
| Type | When to use |
|------|------------|
| HuggingFace Local | Private data, offline, free |
| OpenAI | Best quality, but costs money |

---

### **3. Structured Output Methods (Folder 4)**
| Method | Best for | Validation |
|--------|----------|-----------|
| JSON Schema | Web APIs, flexible | Minimal |
| Pydantic | Python apps, strict types | Excellent |
| TypedDict | Simple hints, lightweight | Medium |

---

### **4. Output Parsers (Folder 5)**
| Parser | Use case | Complexity |
|--------|----------|-----------|
| JSON | Need dictionary/JSON data | Medium |
| Pydantic | Need validated typed data | High |
| String | Just need text | Low |
| Structured | Define specific fields | Medium |

---

## 📝 Learning Path

**Beginner → Advanced:**
1. **Start Here:** Folder 1 (Chat Models) - Just ask questions
2. **Then:** Folder 2 (Prompting) - Get user input
3. **Then:** Folder 3 (Embeddings) - Understand text as numbers
4. **Then:** Folder 5 (Output Parsers) - Organize responses
5. **Master:** Folder 4 (Structured Output) - Professional data handling

---

## 🔑 Key Concepts Recap

| Concept | Means | Example |
|---------|-------|---------|
| **LLM** | Large Language Model | GPT, Gemini, Llama |
| **Embedding** | Text as numbers | [0.25, -0.45, 0.89...] |
| **Chain** | Connected steps (using \|) | template \| model \| parser |
| **Parser** | Cleans AI response | JSON, Pydantic |
| **Prompt Template** | Template with variables | "Tell me about {topic}" |
| **API Key** | Secret password for services | Stored in .env file |
| **Schema** | Structure of expected data | What fields, what types |

---

## 💡 Pro Tips

1. **Always use .env file** for API keys (never hardcode them!)
2. **Start with free models** (Gemma, TinyLlama) before paying
3. **Use Pydantic** when data validation matters
4. **Chain operations** with `|` for cleaner code
5. **Test embeddings** silently (they're fast and local after download)

---

## 🚀 Next Steps to Practice

1. Modify one of the prompts to ask about something you like
2. Add your own fields to a Pydantic class
3. Create a chain that: Report → Summary → Extract key points
4. Build a Streamlit app with your own Pydantic model

Happy Learning! 🎉
