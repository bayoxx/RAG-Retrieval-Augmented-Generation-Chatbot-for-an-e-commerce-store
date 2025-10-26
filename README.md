# RAG-Retrieval-Augmented-Generation-Chatbot-for-an-e-commerce-store
Intelligent AI Customer Support Powered by n8n, Supabase, and Groq


🧠 Project Overview

Cypril Stores RAG Chatbot (Anna) is an AI-powered assistant designed to answer customer inquiries about Cypril Stores — a retail chain operating in Lagos and Abuja, Nigeria.

This chatbot leverages a Retrieval-Augmented Generation (RAG) pipeline built using n8n, Supabase, Cohere, and Groq to provide accurate, context-aware responses sourced directly from Cypril Stores’ business documentation.

🚀 Key Features

💬 Conversational AI: Built with the Groq Chat Model for natural, fast reasoning.

🧾 Knowledge-grounded responses: Uses Supabase Vector Store for document retrieval and context injection.

🧩 Vector Embeddings: Uses Cohere’s Embed-English-v3.0 (1024 Dimensions) for semantic understanding.

📚 Postgres Chat Memory: Tracks user conversations persistently with Supabase’s PostgreSQL table n8n_chat_histories.

⚡ Automated Document Ingestion: Automatically retrains the knowledge base whenever a new business document is uploaded to Google Drive.

🔒 RAG Integrity: Responses are restricted to verified data in the Supabase vector store — no hallucinations.

🌐 Deployed via n8n Production Environment: End users can chat directly with Anna, Cypril Stores’ virtual assistant.

🧩 Architecture Overview
graph TD
    A[Google Drive Trigger] --> B[Download File]
    B --> C[Default Data Loader]
    C --> D[Recursive Character Text Splitter]
    D --> E[Cohere Embeddings (Embed-English-v3.0)]
    E --> F[Supabase Vector Store]
    F -->|Store Embeddings| G[(Supabase Database)]
    
    subgraph Chat Pipeline
    H[User Chat Message] --> I[AI Agent: Cypril Support (Anna)]
    I --> J[Postgres Chat Memory (Supabase)]
    I --> K[Answer with Vector Store]
    K --> L[Groq Chat Model]
    L --> F
    end

🧰 Tools & Technologies
Tool	Purpose
n8n	Automation & workflow orchestration
Supabase	Database + vector storage for RAG
Cohere Embed-English-v3.0 (1024D)	Text embeddings
Groq Chat Model	Fast LLM inference for reasoning
PostgreSQL	Memory storage (chat histories)
Google Drive	Document upload trigger
pgvector	Vector similarity search in SQL
🧮 Supabase Setup

Run the following SQL commands to configure the vector store for embeddings:

-- Enable pgvector extension
CREATE EXTENSION vector;

-- Create table to store document embeddings
CREATE TABLE documents (
  id BIGSERIAL PRIMARY KEY,
  content TEXT,
  metadata JSONB,
  embedding VECTOR(1024)
);

-- Function to perform similarity search
CREATE FUNCTION match_documents (
  query_embedding VECTOR(1024),
  match_count INT DEFAULT NULL,
  filter JSONB DEFAULT '{}'
)
RETURNS TABLE (
  id BIGINT,
  content TEXT,
  metadata JSONB,
  similarity FLOAT
)
LANGUAGE plpgsql
AS $$
#variable_conflict use_column
BEGIN
  RETURN QUERY
  SELECT
    id,
    content,
    metadata,
    1 - (documents.embedding <=> query_embedding) AS similarity
  FROM documents
  WHERE metadata @> filter
  ORDER BY documents.embedding <=> query_embedding
  LIMIT match_count;
END;
$$;

⚙️ Workflow Configuration (n8n)
🔹 1. Document Processing Segment

Trigger: Google Drive → File Created

Embedding Batch Size: 200

Embedding Model: Cohere embed-english-v3.0 (1024D)

Text Splitter: Chunk Size = 1000, Overlap = 200

Destination: Supabase Vector Store → documents table

🔹 2. Chatbot Reasoning Segment

Model: Groq Chat Model

Memory: Postgres Chat Memory (n8n_chat_histories)

Context Window Length: 10

AI Agent Name: Anna (Cypril Stores Customer Support)

Prompt:

You are "Cypril Stores Customer Support," a helpful, polite, and professional AI assistant.
Your primary role is to answer customer inquiries about Cypril Stores' locations, operating hours,
pricing, policies, and services.

Core Rules:
1. Use only information from the Supabase RAG data.
2. Cypril Stores operates in Lagos and Abuja, Nigeria. Prices are in Naira (₦).
3. If unsure, respond with: 
   "I apologize, but I do not have that specific information in my current records.
    Please contact customer service at support@cyprilstores.ng for assistance."
4. Maintain a warm, professional tone at all times.

🔹 3. Vector Retrieval Segment

Node: “Answer questions with a vector store”

Description: "Detailed information regarding Cypril Stores' locations (Lagos and Abuja), business hours, product pricing in Naira (₦), delivery policy, return policy, and loyalty program details."

Limit: 15

Vector Source: Supabase Vector Store → Table: documents

Connected Model: Groq Chat Model

🧾 Business Knowledge Base

The AI model was trained using the Cypril Stores Business Document, which includes:

Store locations, hours, and contacts (Lagos & Abuja)

Product categories, unit prices, and discounts

Delivery, return, and refund policies

Loyalty program (Cypril Rewards Club)

Internal operational notes (manager contacts, peak hours)

💬 Example Chat Flow

User:

Hi, what are your operating hours at the Wuse branch?

Anna:

The Wuse (Abuja) branch operates daily from 8:00 AM to 8:00 PM.
You can reach the store at 080-CYPRIL-A.

🧠 Memory Example

The chatbot stores recent interactions (last 10 exchanges) in the n8n_chat_histories table using Supabase PostgreSQL, enabling contextual continuity within a session.

🌍 Deployment

Platform: n8n Cloud (Production)

Access: Chatbot accessible via custom chat URL.

First Message:

Hi there! 👋
My name is Anna. Welcome to Cypril Stores. How can I assist you today?

📸 Workflow Snapshot

🧩 Future Improvements

Integrate live inventory data via Cypril’s internal API

Add sentiment analysis node for customer feedback

Enable multilingual support (English + Yoruba)

Deploy to WhatsApp or website widget via n8n Webhook

📚 References

n8n Documentation

Supabase Vector Database

Cohere Embeddings API

Groq AI Models

👩🏽‍💻 Author

Bayo Yusuff
Data & Automation Analyst | AI Workflow Developer
📧 support@cyprilstores.ng

🌐 GitHub Portfolio
