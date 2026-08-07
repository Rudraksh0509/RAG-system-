# n8n RAG AI Agent --- Google Drive + Gemini + Pinecone

A **Retrieval-Augmented Generation (RAG)** workflow built with **n8n**
that converts documents stored in Google Drive into a searchable AI
knowledge base. Users can ask questions through the n8n chat interface,
and the AI Agent retrieves relevant information from Pinecone before
Google Gemini generates the final response.

## 🚀 Features

-   Google Drive document ingestion
-   Automatic document loading and text extraction
-   Recursive text chunking
-   Google Gemini embeddings
-   Pinecone vector storage and semantic retrieval
-   Google Gemini Chat Model
-   n8n AI Agent
-   Conversation memory
-   Document-grounded question answering

## 🏗️ Workflow Architecture

The workflow is divided into two connected systems:

1.  **Document Ingestion Pipeline** --- prepares documents and stores
    their knowledge in Pinecone.
2.  **RAG Question-Answering Pipeline** --- retrieves relevant knowledge
    and uses it to answer user questions.

### Document Ingestion

``` text
Manual Trigger
    ↓
Search Files and Folders
    ↓
Download File
    ↓
Default Data Loader
    ↓
Recursive Character Text Splitter
    ↓
Google Gemini Embeddings
    ↓
Pinecone Vector Store
```

### Question Answering

``` text
User
 ↓
Chat Trigger
 ↓
AI Agent
 ├── Google Gemini Chat Model
 ├── Simple Memory
 └── Vector Store Tool
          ↓
     Pinecone Vector Store
          ↓
     Gemini Embeddings
          ↓
   Relevant Document Chunks
          ↓
       AI Agent
          ↓
     Final Response
```

## 🔍 Workflow Explanation

### 1. Manual Trigger

The ingestion side begins with **When clicking "Execute workflow"**.
This gives the developer manual control over when documents are
processed and indexed.

It is useful during development because embedding documents consumes API
calls and normally does not need to happen every time a user asks a
question.

### 2. Search Files and Folders --- Google Drive

The workflow searches Google Drive for the document or files that should
become part of the knowledge base.

Google Drive therefore acts as the **document source** for the RAG
system.

Example sources include PDFs, lecture notes, research papers,
documentation, reports, or other knowledge files.

### 3. Download File

After the required file is found, the **Download File** node retrieves
the actual document from Google Drive.

The downloaded file is then passed into the document-processing
components.

### 4. Default Data Loader

The **Default Data Loader** converts the downloaded file into document
content that the AI pipeline can process.

Conceptually:

``` text
Raw File → Data Loader → Extracted Document Text
```

This bridges the gap between a stored file and the text required for
chunking and embeddings.

### 5. Recursive Character Text Splitter

Large documents should not normally be represented by a single vector.
The text splitter therefore divides the document into smaller, partially
independent **chunks**.

``` text
Document
 ├── Chunk 1
 ├── Chunk 2
 ├── Chunk 3
 └── Chunk N
```

Smaller chunks improve retrieval because Pinecone can return the
specific section relevant to a question rather than an entire document.

A recursive splitter also attempts to preserve natural text boundaries
where possible, helping maintain useful context inside each chunk.

### 6. Google Gemini Embeddings

Each text chunk is passed through **Google Gemini Embeddings**.

An embedding converts natural language into a numerical vector
representing semantic meaning.

``` text
"RAG combines retrieval with generative AI"
                    ↓
             Gemini Embeddings
                    ↓
        [0.18, -0.42, 0.71, ...]
```

Semantically similar text produces vectors that are relatively close in
vector space. This enables searches based on **meaning**, not just exact
keyword matches.

### 7. Pinecone Vector Store --- Indexing

The generated vectors are stored in **Pinecone** together with the
corresponding document content and available metadata.

At this point, the original document has effectively become a searchable
semantic knowledge base.

The expensive document-processing stage only needs to run when documents
need to be added or updated.

------------------------------------------------------------------------

## 💬 RAG Question-Answering Pipeline

The second half of the workflow handles conversations with the user.

### 8. Chat Trigger

The **When chat message received** node starts the conversational
pipeline whenever a user sends a message through the n8n chat interface.

Example:

``` text
User: "What are the main topics discussed in the document?"
```

The message is passed to the AI Agent.

### 9. AI Agent

The **AI Agent** is the central orchestration component of the chatbot.

It is connected to three important capabilities:

``` text
AI Agent
 ├── Gemini Chat Model → reasoning and response generation
 ├── Simple Memory → conversation context
 └── Vector Store Tool → document retrieval
```

Instead of simply sending every question directly to an LLM, the agent
can use the vector-store tool to retrieve relevant knowledge first.

### 10. Answer Questions with a Vector Store

This tool gives the AI Agent access to the RAG knowledge base.

When document knowledge is required, the user's question is used to
search the vector store.

For example:

``` text
"What does the document say about vector databases?"
                         ↓
                  Vector Store Tool
                         ↓
                     Pinecone
```

### 11. Query Embedding

The user's question is converted into an embedding using the
retrieval-side **Google Gemini Embeddings** node.

``` text
User Question
     ↓
Gemini Embeddings
     ↓
Query Vector
```

The query vector can then be mathematically compared with the document
vectors already stored in Pinecone.

### 12. Pinecone Similarity Search

Pinecone searches for document chunks whose vectors are closest to the
query vector.

Conceptually:

``` text
Question Vector
      ↓
Pinecone Similarity Search
      ↓
Most Relevant Chunks
```

If a document contains several unrelated sections, only the chunks most
relevant to the question need to be returned.

### 13. Retrieved Context + Gemini

The retrieved document information is returned to the AI Agent.

The agent can then use Gemini to generate an answer using both:

``` text
User Question
      +
Retrieved Document Context
      +
Conversation Context
      ↓
Google Gemini
      ↓
Final Answer
```

This is the key difference between a normal chatbot and this RAG
workflow.

A normal chatbot primarily depends on the LLM's existing knowledge. This
system can additionally ground answers in the documents stored in the
private knowledge base.

### 14. Simple Memory

**Simple Memory** allows the agent to maintain context across messages
in a conversation.

Example:

``` text
User: Explain RAG.
AI: RAG combines retrieval with generation.

User: What are its advantages?
```

Memory helps the agent understand that **"its"** refers to RAG rather
than treating the second message as an unrelated question.

------------------------------------------------------------------------

## 🧠 Complete Data Flow

### Indexing Phase

``` text
Google Drive
     ↓
Find Document
     ↓
Download Document
     ↓
Extract Text
     ↓
Split into Chunks
     ↓
Generate Gemini Embeddings
     ↓
Store Vectors in Pinecone
```

### Retrieval Phase

``` text
User Question
     ↓
AI Agent
     ↓
Generate Query Embedding
     ↓
Search Pinecone
     ↓
Retrieve Relevant Chunks
     ↓
Provide Context to Gemini
     ↓
Generate Grounded Answer
     ↓
Return Response to User
```

## 🛠️ Technology Stack

  Component                     Purpose
  ----------------------------- ------------------------------------------
  **n8n**                       Workflow automation and AI orchestration
  **Google Drive**              Source document storage
  **Google Gemini**             LLM for reasoning and answer generation
  **Gemini Embeddings**         Converts text and queries into vectors
  **Pinecone**                  Vector database and semantic search
  **AI Agent**                  Coordinates model, retrieval, and memory
  **Simple Memory**             Maintains conversational context
  **Recursive Text Splitter**   Creates retrievable document chunks

## ⚙️ Setup

### Prerequisites

You need:

-   n8n Cloud or self-hosted n8n
-   Google Drive access
-   Google Gemini API credentials
-   Pinecone account and vector index

### Configuration

1.  Import the workflow JSON into n8n.
2.  Configure Google Drive credentials.
3.  Configure Google Gemini credentials for the Chat Model and
    Embeddings.
4.  Configure Pinecone credentials and select the appropriate index.
5.  Ensure the Pinecone index is compatible with the embedding model
    configuration.
6.  Select the Google Drive document/folder to index.
7.  Execute the ingestion pipeline.
8.  Open the n8n chat and ask questions about the indexed documents.

> **Security:** Never upload API keys, OAuth tokens, Pinecone
> credentials, or other secrets to GitHub.

## 💬 Example Questions

``` text
Summarize the uploaded document.

What are the key concepts discussed in the notes?

Explain the main topic in simple language.

What does the document say about RAG?

Compare two concepts mentioned in the document.
```

## 📂 Recommended Repository Structure

``` text
n8n-rag-ai-agent/
├── README.md
├── LICENSE
├── .gitignore
├── workflow/
│   └── rag-ai-agent.json
└── screenshots/
    └── workflow.png
```

Add the workflow image to the README with:

``` markdown
![n8n RAG Workflow](screenshots/workflow.png)
```

## 🔐 Security

Recommended `.gitignore`:

``` gitignore
.env
.env.*
credentials.json
secrets.json
node_modules/
```

Store credentials in n8n's credential manager or environment variables.

## 🔮 Future Improvements

-   Automatically index new Google Drive files
-   Return source document citations with answers
-   Add document metadata and filtering
-   Support multiple folders and document types
-   Add hybrid keyword + semantic retrieval
-   Add reranking for better context selection
-   Connect the workflow to a custom web application
-   Add RAG evaluation for retrieval relevance and answer faithfulness

## 📌 Use Cases

This architecture can be adapted for **PDF chatbots, AI study
assistants, research assistants, internal knowledge bases, documentation
Q&A systems, college notes assistants, and private document search
systems**.

## 📄 License

This project can be released under the **MIT License**.

------------------------------------------------------------------------

### Core Architecture

**Google Drive → Document Processing → Gemini Embeddings → Pinecone →
Semantic Retrieval → n8n AI Agent → Gemini → Context-Aware Answer**
