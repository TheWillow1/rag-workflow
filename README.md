# RAG Document Chat System

A Retrieval-Augmented Generation (RAG) workflow built with **n8n** that allows users to upload documents and interact with their content through an AI-powered chat interface.

The system separates the process into two main workflows:

1. **Document Ingestion** — processes and stores uploaded documents.
2. **Question & Answer** — retrieves relevant information from the stored documents and uses an AI Agent to generate responses.

The workflow is designed to work with a frontend application that communicates with n8n through webhooks.


# Architecture

```text
                 DOCUMENT INGESTION
                        
        ┌──────────────────────────┐
        │      Frontend App        │
        │    Document Upload       │
        └────────────┬─────────────┘
                     │
                     ▼
             ┌───────────────┐
             │ Document      │
             │ Upload       │
             │ Webhook       │
             └───────┬───────┘
                     │
                     ▼
             ┌───────────────┐
             │ Default Data  │
             │ Loader        │
             └───────┬───────┘
                     │
                     ▼
             ┌───────────────┐
             │ Google Gemini │
             │ Embeddings    │
             └───────┬───────┘
                     │
                     ▼
             ┌───────────────┐
             │ Insert Data   │
             │ to Store      │
             └───────────────┘


                 QUESTION & ANSWER

        ┌──────────────────────────┐
        │      Frontend Chat       │
        └────────────┬─────────────┘
                     │
                     ▼
             ┌───────────────┐
             │ Chat Webhook  │
             └───────┬───────┘
                     │
                     ▼
             ┌───────────────┐
             │   AI Agent    │
             └───┬─────┬─────┘
                 │     │
        ┌────────┘     └──────────┐
        ▼                         ▼
┌───────────────┐         ┌───────────────┐
│ Query Data    │         │ Simple Memory │
│ Tool          │         │               │
└───────┬───────┘         └───────────────┘
        │
        ▼
┌───────────────┐
│ Gemini        │
│ Embeddings    │
└───────────────┘

                 │
                 ▼
        ┌────────────────┐
        │ OpenRouter     │
        │ Chat Model     │
        └───────┬────────┘
                │
                ▼
        ┌────────────────┐
        │ Response to    │
        │ Frontend       │
        └────────────────┘
```


# 1. Document Upload

### Node: `Document Upload`

The document ingestion process begins with a **Webhook**.

The frontend sends an uploaded document to this webhook, allowing n8n to receive the document and begin processing it.

### Purpose

* Receive documents from the frontend
* Start the document-processing pipeline
* Pass the uploaded document to the data-loading stage


# 2. Document Processing

### Node: `Default Data Loader`

The **Default Data Loader** processes the incoming document and prepares its contents for storage.

This converts the uploaded document into data that can be processed by the RAG pipeline.

The output is then passed toward the vector-storage process.


# 3. Generate Embeddings

### Node: `Embeddings Google Gemini`

The workflow uses **Google Gemini embeddings** to convert document content into numerical vector representations.

These embeddings allow the system to represent the semantic meaning of the document content in a form that can later be searched.

This is an important part of the RAG architecture because the system isn't simply searching for exact keywords.

It can use the semantic representation of the content when looking for relevant information.


# 4. Store Document Data

### Node: `Insert Data to Store`

The processed document data and its embeddings are inserted into the workflow's vector store.

This creates the knowledge base that the AI Agent can query later.

The ingestion pipeline therefore follows:

```text
Document
   ↓
Data Loader
   ↓
Embeddings
   ↓
Vector Store
```

Once this process is complete, the document's information is available for retrieval.


# 5. Receive Chat Messages

### Node: `Chat`

The second part of the workflow begins with another **Webhook**.

The frontend sends user chat messages to this webhook.

For example:

```text
"What does this document say about...?"
```

The webhook passes the user's request to the AI Agent.


# 6. AI Agent

### Node: `AI Agent`

The **AI Agent** is the central component of the question-answering process.

It receives the user's message and has access to several supporting components:

* Query Data Tool
* Simple Memory
* OpenRouter Chat Model

The Agent can use the available tools and context to formulate an appropriate response.

This is where the workflow moves from simple automation into an AI-powered retrieval system.


# 7. Query the Knowledge Base

### Node: `Query Data Tool`

The Query Data Tool provides the AI Agent with access to the stored document information.

When the Agent needs information from the uploaded documents, it can use this tool to search the vector store.

The general retrieval process is:

```text
User Question
      ↓
AI Agent
      ↓
Query Data Tool
      ↓
Vector Store
      ↓
Relevant Information
      ↓
AI Agent
```

The retrieved information gives the AI additional context for generating its response.


# 8. Conversation Memory

### Node: `Simple Memory`

The workflow also includes a **Simple Memory** component.

Its purpose is to maintain conversational context between messages.

This allows the AI Agent to handle follow-up questions more naturally instead of treating every message as an isolated request.

For example:

```text
User:
"What is this document about?"

AI:
"The document discusses..."

User:
"What does it say about citations?"

```

The memory component allows the Agent to retain the context of the previous interaction.


# 9. Chat Model

### Node: `OpenRouter Chat Model`

The AI Agent uses an **OpenRouter Chat Model** to generate the final response.

The model receives the user's request along with the relevant information retrieved from the knowledge base.

The basic process is:

```text
User Question
      +
Retrieved Context
      ↓
Chat Model
      ↓
Generated Answer
```


# 10. Return the Response

### Node: `Respond to Webhook1`

After the AI Agent generates its response, the result is returned through the webhook response node.

This allows the frontend application to receive the AI-generated answer and display it to the user.

The complete chat flow is therefore:

```text
Frontend
   ↓
Chat Webhook
   ↓
AI Agent
   ↓
Query Knowledge Base
   ↓
Retrieve Relevant Context
   ↓
Chat Model
   ↓
AI Response
   ↓
Frontend
```


# 🔄 Complete RAG Pipeline

The entire system can be summarized in two pipelines.

### Document Ingestion

```text
Frontend
    ↓
Document Upload Webhook
    ↓
Default Data Loader
    ↓
Google Gemini Embeddings
    ↓
Vector Store
```

### Question & Answer

```text
Frontend
    ↓
Chat Webhook
    ↓
AI Agent
    ↓
Query Data Tool
    ↓
Vector Store
    ↓
Relevant Context
    ↓
AI Agent + Chat Model
    ↓
Webhook Response
    ↓
Frontend
```


# Technologies & Components

| Component                    | Purpose                                                     |
| ---------------------------- | ----------------------------------------------------------- |
| **n8n**                      | Workflow orchestration                                      |
| **Webhooks**                 | Communication between frontend and backend                  |
| **Google Gemini Embeddings** | Converts document content into vector representations       |
| **Vector Store**             | Stores and retrieves document information                   |
| **AI Agent**                 | Controls the question-answering process                     |
| **Query Data Tool**          | Allows the Agent to search the knowledge base               |
| **Simple Memory**            | Maintains conversational context                            |
| **OpenRouter Chat Model**    | Generates AI responses                                      |
| **Lovable Frontend**         | Provides the user-facing document upload and chat interface |

---

# What This Project Demonstrates

This project demonstrates practical experience with:

* RAG architecture
* AI Agents
* Vector embeddings
* Vector search
* Document processing
* Webhook-based system integration
* LLM integration
* Conversational memory
* Frontend-to-backend communication
* n8n workflow orchestration
* Building AI applications rather than isolated AI workflows

