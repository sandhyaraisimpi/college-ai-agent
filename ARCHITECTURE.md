📄 College AI Agent – Architecture Design
-- Overview

This document describes the architecture for an AI agent designed for a college website assistant. The agent helps students, parents, and staff by handling:

Admission-related queries
Course and department information
Fees and scholarship details
Academic and administrative policies
Important dates and schedules

The agent strictly retrieves information from official structured databases and institutional documents, ensuring accuracy and reliability.

-- High-Level Architecture

The system consists of three main layers:


User (Web Chat / Mobile App)
        |
        v
   [FastAPI Backend]  - handles authentication, sessions, request validation
        |
        v
   [AI Agent / Orchestrator (LangChain)]
        |
        +--> Query Understanding (intent + entity extraction)
        +--> Reasoning Engine (decision making)
        +--> Tool Selection (which data source to use)
        +--> Tool Execution (fetch structured/unstructured data)
        +--> Response Generation
        |
        v
   [Data Layer]
        +--> Admission Database
        +--> Fee & Scholarship Tables
        +--> Department & Course DB
        +--> Policy Documents (PDFs)
        +--> Academic Calendar


       
        -- Internal Components
--- 1. Query Understanding (Intent Detection)

The AI agent uses an LLM to classify queries into categories:

ADMISSION_QUERY
ACADEMIC_QUERY
FINANCE_QUERY (fees/scholarships)
POLICY_QUERY
GENERAL_QUERY

Example:

“What is the MBA fee?” → FINANCE_QUERY

The LLM also extracts entities like:

Course name
Department
Date
--- 2. Reasoning Engine (Core AI Behavior ⭐)

This is the most important component.

The agent decides:

Is the query complete?
Do we need clarification?
Which data source is required?
Structured DB or document retrieval?
Is retrieved information sufficient?

Example:

“Tell me fees” → Needs clarification → asks user
“B.Tech fees” → Direct retrieval


--- 3. Router (Decision Point)

Based on intent, the agent selects tools:

Query Type	     Tool
Admission    	admission_tool
Fees	        finance_tool
Courses	       academic_tool
Policies	   policy_tool
Dates	       calendar_tool



--- 4. Tools (Data Access Layer)

Each tool is a Python function:

admission_tool → admission rules, eligibility
finance_tool → fee tables, scholarship details
academic_tool → courses, departments
policy_tool → extracts info from PDFs
calendar_tool → deadlines, events


--- 5. Knowledge Retrieval

Two types:

Structured Data:
SQL queries on database
Unstructured Data:
RAG (Retrieval-Augmented Generation)
Uses embeddings to search documents



-- 6. Clarification Handler

If query is incomplete:

Example:

User: “Tell me fees”
→ Agent: “Which course are you asking about?”


--- 7. Validation Layer

Ensures:

Data is from official sources
Response is accurate
No hallucinated information



--- 8. Response Generator

Converts raw data into natural language:

Example:

{"course": "B.Tech", "fee": "₹1,20,000/year"}

→
“B.Tech fee is ₹1,20,000 per year.”

-- End-to-End Flow Example
Query:

“Tell me B.Tech admission eligibility

-- End-to-End Flow Example
Query:

“Tell me B.Tech admission eligibility”

Step 1 – Intent Detection

→ ADMISSION_QUERY

Step 2 – Reasoning

→ Query is complete
→ No clarification needed

Step 3 – Tool Selection

→ admission_tool

Step 4 – Data Retrieval

→ Fetch eligibility from database

Step 5 – Validation

→ Verify correctness

Step 6 – Response

→ “B.Tech requires 10+2 with PCM and minimum 75% marks.”

-- Tech Stack

| Component        | Technology                | Why                               |
| ---------------- | ------------------------- | --------------------------------- |
| LLM              | OpenAI GPT-4o             | Strong reasoning + NLP            |
| Agent Framework  | LangChain                 | Handles agent flow, tools, memory |
| Backend          | FastAPI                   | Fast, scalable APIs               |
| Database         | MySQL / PostgreSQL        | Structured data storage           |
| Vector DB        | FAISS / Pinecone          | Document search                   |
| ORM              | SQLAlchemy                | Easy DB integration               |
| Document Parsing | PyPDF / LangChain loaders | Extract PDF data                  |
| Memory           | ConversationBufferMemory  | Context awareness                 |

-- Flow Diagram

+----------------------+
|      User Query      |
| (Web / Mobile Chat)  |
+----------+-----------+
           |
           v
+---------------------------+
|   Input Processing Layer  |
| (Cleaning, Tokenization)  |
+-----------+---------------+
            |
            v
+---------------------------+
|  Query Understanding      |
| (Intent + Entity Extract) |
+-----------+---------------+
            |
            v
+---------------------------+
|   Reasoning Engine        |
|---------------------------|
| • Is query complete?      |
| • Need clarification?     |
| • Select data source      |
+-----+-------------+-------+
      |             |
      | Yes         | No
      v             v
+-------------+   +--------------------+
| Ask User    |   |     Router         |
| Clarification|  | (Select Tool)      |
+-------------+   +---------+----------+
                           |
                           v
              +---------------------------+
              |     Tool Selection        |
              |---------------------------|
              | admission_tool            |
              | finance_tool              |
              | academic_tool             |
              | policy_tool               |
              | calendar_tool             |
              +-----------+---------------+
                          |
                          v
              +---------------------------+
              |   Knowledge Retrieval     |
              |---------------------------|
              | Structured (SQL DB)       |
              | Unstructured (RAG/PDFs)   |
              +-----------+---------------+
                          |
                          v
              +---------------------------+
              |     Validation Layer      |
              |---------------------------|
              | • Source check            |
              | • Completeness check      |
              | • Accuracy check          |
              +-----------+---------------+
                          |
                          v
              +---------------------------+
              |   Response Generator      |
              | (Natural Language Output) |
              +-----------+---------------+
                          |
                          v
              +---------------------------+
              |     Final Response        |
              |      to User              |
              +---------------------------+


          -- Key Design Decisions
✅ Single LLM Approach
One model handles classification + reasoning
Reduces complexity
✅ Dynamic Routing
Queries routed based on intent
✅ Hybrid Retrieval
Structured + unstructured data
✅ Clarification First
Avoids wrong answers
✅ Validation Layer
Prevents hallucination


-- Why This is an AI Agent

This system is not a basic chatbot because:

It reasons before answering
It decides dynamically what action to take
It asks follow-up questions
It uses multiple data sources
It validates responses before output