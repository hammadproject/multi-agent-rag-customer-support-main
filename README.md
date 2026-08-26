# Multi-Agent RAG Customer Support

A modular customer-support AI system that combines **LangGraph**, **LangChain**, **OpenAI**, and **Qdrant** to handle travel-related requests through a coordinated set of specialized assistants.

The system uses Retrieval-Augmented Generation (RAG) to search structured travel information and routes each request to the assistant best suited to the task.

https://github.com/hammadproject/multi-agent-rag-customer-support-main/blob/main/images/multi_agent_rag_system_architecture_aws.png?raw=true

## Overview

The application is designed around a supervisor-and-specialists architecture.

A **Primary Assistant** receives the user's request and delegates specialized work to dedicated assistants for:

- Flight booking
- Car rental
- Hotel booking
- Excursions and trip recommendations

The assistants can use application tools to search and work with travel data. Operations considered sensitive can pause the workflow and request user confirmation before continuing.

The project is split into two main components:

1. **Vectorizer** — prepares source data, creates embeddings, and indexes content in Qdrant.
2. **Customer Support Chat** — runs the multi-agent conversation workflow and retrieves relevant information from the vector database.

---
https://github.com/hammadproject/multi-agent-rag-customer-support-main/blob/main/images/travel_db_schema.png?raw=true

## Core Workflow

```text
User Request
     │
     ▼
Primary Assistant
     │
     ├──────────────┬──────────────┬──────────────┐
     ▼              ▼              ▼              ▼
   Flights       Car Rental      Hotels       Excursions
     │              │              │              │
     └──────────────┴──────────────┴──────────────┘
                            │
                            ▼
                       RAG / Tools
                            │
                  ┌─────────┴─────────┐
                  ▼                   ▼
              Safe Action       Sensitive Action
                                      │
                                      ▼
                              User Confirmation
                                      │
                                      ▼
                                  Final Reply
```

---

## Key Features

### Multi-Agent Routing

A primary assistant acts as the entry point and delegates specialized requests to dedicated assistants.

This keeps domain-specific logic separated and makes the workflow easier to extend.

### Specialized Travel Assistants

The current implementation contains:

- **Flight Booking Assistant** — handles flight-related requests.
- **Car Rental Assistant** — handles vehicle rental requests.
- **Hotel Booking Assistant** — handles hotel-related requests.
- **Excursion Assistant** — handles excursion and trip recommendations.

### Retrieval-Augmented Generation

The system uses Qdrant as the vector database for semantic retrieval.

Travel information is processed into chunks, converted into embeddings, and indexed into collections that can be searched by the customer-support workflow.

### Tool-Based Actions

Assistants can use tools for domain-specific operations, including:

- Flight lookup
- Hotel lookup
- Car lookup
- Excursion lookup
- General travel-data lookup

### Sensitive-Action Confirmation

The LangGraph workflow distinguishes between normal tool usage and operations that require user approval.

When confirmation is required, execution can pause before the sensitive action is completed.

### Conversation State

The workflow maintains state while the request moves between the primary assistant, specialized assistants, and tools.

A memory checkpointer is used by the graph to preserve workflow progress.

### LangSmith Observability

The project supports LangSmith tracing through environment configuration, allowing developers to inspect:

- Agent execution
- Tool calls
- Request traces
- Errors
- Workflow behavior

LangSmith is optional for local execution.

---

## Architecture

### Application Layer

```text
customer_support_chat/
    │
    ├── core/
    │   ├── logger.py
    │   ├── settings.py
    │   └── state.py
    │
    ├── graph.py
    ├── main.py
    │
    └── services/
        ├── assistants/
        ├── tools/
        └── vectordb/
```

The `graph.py` module builds the LangGraph workflow, while the assistant modules contain the specialized conversational logic.

### Vectorization Layer

```text
vectorizer/
    │
    └── app/
        ├── core/
        ├── embeddings/
        ├── vectordb/
        └── main.py
```

The vectorizer reads source data, splits content into chunks, generates embeddings through the OpenAI API, and indexes those embeddings in Qdrant.

---

## RAG Pipeline

```text
Travel / FAQ Data
       │
       ▼
Content Formatting
       │
       ▼
Recursive Text Splitting
       │
       ▼
OpenAI Embeddings
       │
       ▼
Qdrant Collections
       │
       ▼
Semantic Search
       │
       ▼
Relevant Context
       │
       ▼
Specialized Assistant
       │
       ▼
Final Response
```

The vectorizer supports separate collections for travel-related content such as flights, hotels, car rentals, trips, and FAQs.

---

## Technology Stack

| Technology | Purpose |
|---|---|
| Python 3.12+ | Application runtime |
| LangGraph | Multi-agent workflow orchestration |
| LangChain | Agent and tool framework |
| OpenAI API | LLM and embedding generation |
| Qdrant | Vector database |
| SQLite | Source travel database |
| Pandas | Data processing |
| aiohttp | Asynchronous API requests |
| DuckDuckGo Search | Web-search integration dependency |
| LangChain Community | Additional integrations |
| Google Community Integrations | Google-related LangChain integrations |
| Poetry | Dependency and environment management |
| Docker | Containerized execution |
| LangSmith | Optional tracing and observability |

---

## Project Structure

The repository currently contains:

```text
.
├── .dev.env
├── .gitignore
├── .vscode/
│   └── launch.json
├── Dockerfile
├── Makefile
├── README.md
├── docker-compose.yml
├── poetry.lock
├── pyproject.toml
│
├── customer_support_chat/
│   ├── README.md
│   ├── __init__.py
│   ├── app/
│   │   ├── __init__.py
│   │   ├── core/
│   │   │   ├── __init__.py
│   │   │   ├── logger.py
│   │   │   ├── settings.py
│   │   │   └── state.py
│   │   ├── graph.py
│   │   ├── main.py
│   │   └── services/
│   │       ├── __init__.py
│   │       ├── assistants/
│   │       │   ├── __init__.py
│   │       │   ├── assistant_base.py
│   │       │   ├── car_rental_assistant.py
│   │       │   ├── excursion_assistant.py
│   │       │   ├── flight_booking_assistant.py
│   │       │   ├── hotel_booking_assistant.py
│   │       │   └── primary_assistant.py
│   │       ├── tools/
│   │       │   ├── __init__.py
│   │       │   ├── cars.py
│   │       │   ├── excursions.py
│   │       │   ├── flights.py
│   │       │   ├── hotels.py
│   │       │   └── lookup.py
│   │       ├── utils.py
│   │       └── vectordb/
│   │           ├── __init__.py
│   │           ├── chunkenizer.py
│   │           ├── utils.py
│   │           └── vectordb.py
│   │
│   └── data/
│       └── user_test_fetch_data.sql
│
├── vectorizer/
│   ├── README.md
│   ├── __init__.py
│   └── app/
│       ├── core/
│       │   ├── __init__.py
│       │   ├── logger.py
│       │   └── settings.py
│       ├── embeddings/
│       │   ├── __init__.py
│       │   └── embedding_generator.py
│       ├── main.py
│       └── vectordb/
│           ├── __init__.py
│           ├── chunkenizer.py
│           ├── utils.py
│           └── vectordb.py
│
├── graphs/
│   └── multi-agent-rag-system-graph.png
│
└── images/
    ├── langsmith.gif
    ├── multi_agent_rag_system_architecture_aws.png
    ├── qdrant_schema.png
    ├── travel_db_schema.png
    └── ytb.png
```

---

## Requirements

The project currently specifies:

- **Python 3.12+**
- **Poetry**
- **Docker / Docker Compose**
- An **OpenAI API key**
- A **LangSmith API key** if tracing is enabled

The dependency configuration is defined in `pyproject.toml`.

---

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/hammadproject/multi-agent-rag-customer-support-main.git
cd multi-agent-rag-customer-support-main
```

### 2. Create the Environment File

Copy the development environment template:

```bash
cp .dev.env .env
```

Then update `.env` with your own credentials and configuration.

### 3. Configure OpenAI

Set:

```env
OPENAI_API_KEY="your_openai_api_key"
```

The same API is used by the vectorizer for embedding generation.

### 4. Configure Qdrant

The default local configuration is:

```env
QDRANT_URL=http://localhost:6333
```

The provided Docker Compose configuration runs Qdrant on port `6333`.

### 5. Configure the Travel Database

The configured SQLite source path is:

```env
SQLITE_DB_PATH=./customer_support_chat/data/travel2.sqlite
```

The repository currently references this path from the application configuration.

### 6. Optional LangSmith Configuration

For tracing, configure:

```env
LANGCHAIN_TRACING_V2="true"
LANGCHAIN_ENDPOINT="https://api.smith.langchain.com"
LANGCHAIN_API_KEY="your_langsmith_api_key"
LANGCHAIN_PROJECT="your_project_name"
```

LangSmith tracing can be omitted if you do not need observability.

---

## Install Dependencies

Install the Poetry dependencies:

```bash
poetry install
```

Activate the Poetry environment if desired:

```bash
poetry shell
```

Or execute commands directly through Poetry:

```bash
poetry run python ...
```

---

## Start Qdrant

Launch the local Qdrant service:

```bash
docker compose up qdrant -d
```

Qdrant will be available on:

```text
http://localhost:6333
```

The Qdrant dashboard is available at:

```text
http://localhost:6333/dashboard
```

---

## Build the Vector Database

The vectorizer generates embeddings and creates the required Qdrant collections.

Run:

```bash
poetry run python vectorizer/app/main.py
```

The vectorizer processes the configured SQLite source and creates embeddings for the supported content collections.

The environment variable:

```env
RECREATE_COLLECTIONS="True"
```

controls whether collections should be recreated according to the vectorizer's configuration.

For large datasets, review `LIMIT_ROWS` before running the process.

---

## Start the Customer Support Chat

After Qdrant and the required collections are available:

```bash
poetry run python ./customer_support_chat/app/main.py
```

The application starts the customer-support interaction loop and routes requests through the LangGraph workflow.

---

## Docker

The repository includes:

```text
Dockerfile
docker-compose.yml
```

The Docker Compose configuration currently defines a Qdrant service and a customer-support application service.

Start the stack with:

```bash
docker compose up --build
```

The application container depends on Qdrant.

For local development, the application configuration and environment variables should be reviewed before using the containerized setup.

---

## Environment Variables

| Variable | Purpose | Required |
|---|---|---|
| `OPENAI_API_KEY` | OpenAI API access and embeddings | Yes |
| `QDRANT_URL` | Qdrant server URL | Yes |
| `SQLITE_DB_PATH` | SQLite travel database path | Yes |
| `LOG_LEVEL` | Application logging level | No |
| `RECREATE_COLLECTIONS` | Controls collection recreation behavior | No |
| `LANGCHAIN_TRACING_V2` | Enables LangSmith tracing | No |
| `LANGCHAIN_ENDPOINT` | LangSmith endpoint | No |
| `LANGCHAIN_API_KEY` | LangSmith authentication | No |
| `LANGCHAIN_PROJECT` | LangSmith project name | No |
| `LIMIT_ROWS` | Limits rows processed during indexing | No |

Never commit real credentials to `.env` or `.dev.env`.

---

## Main Components

### `customer_support_chat/app/graph.py`

Builds the multi-agent state graph.

The graph contains the primary assistant, specialized assistant nodes, tool nodes, routing logic, and interrupt/confirmation handling.

### `primary_assistant.py`

Acts as the supervisor for general user requests and delegates specialized tasks.

### Specialized Assistants

Located under:

```text
customer_support_chat/app/services/assistants/
```

They include:

- `flight_booking_assistant.py`
- `car_rental_assistant.py`
- `hotel_booking_assistant.py`
- `excursion_assistant.py`

### Tools

Located under:

```text
customer_support_chat/app/services/tools/
```

They provide domain-specific operations for:

- Flights
- Hotels
- Cars
- Excursions
- General lookup

### Vector Database Integration

The customer-support application contains Qdrant integration under:

```text
customer_support_chat/app/services/vectordb/
```

The vectorizer has a corresponding implementation under:

```text
vectorizer/app/vectordb/
```

---

## Vectorizer

The vectorizer is responsible for preparing content for semantic search.

### Embedding Generation

`embedding_generator.py` generates embeddings through the OpenAI API and supports both individual strings and lists of strings.

### Text Chunking

`chunkenizer.py` uses LangChain's recursive character splitting approach to divide larger content into smaller chunks.

### Qdrant Indexing

`vectordb.py` handles:

- Qdrant connections
- Collection creation/clearing
- Content formatting
- Embedding generation
- Document indexing
- FAQ indexing
- Vector search

The implementation uses asynchronous processing and batches embedding requests.

---

## Data Sources

The project is configured around a SQLite travel dataset containing travel-related information used by the customer-support tools and vectorization workflow.

The configured database path is:

```text
customer_support_chat/data/travel2.sqlite
```

The repository also contains:

```text
customer_support_chat/data/user_test_fetch_data.sql
```

for database-related testing/query scenarios.

---

## Safety & Human Confirmation

The workflow separates tools according to whether they can be executed directly or require confirmation.

The graph can interrupt execution before a sensitive operation is performed, allowing the user to approve or reject the action.

This pattern is useful for operations that can change booking or customer state and helps prevent unintended actions.

---

## Observability

LangSmith tracing is supported through the environment variables:

```env
LANGCHAIN_TRACING_V2
LANGCHAIN_ENDPOINT
LANGCHAIN_API_KEY
LANGCHAIN_PROJECT
```

When enabled, tracing can help inspect:

- Agent routing
- Tool execution
- Retrieval operations
- Model responses
- Errors
- Workflow execution

LangSmith is optional and is not required for the core local workflow.

---

## Development Commands

The repository includes a small `Makefile`.

Show available commands:

```bash
make help
```

Clean Python cache directories:

```bash
make clean
```

The default Make target displays the available commands.

---

## Configuration & Customization

### Add a New Specialized Assistant

A new domain assistant can be added under:

```text
customer_support_chat/app/services/assistants/
```

The assistant should follow the existing assistant pattern and be connected to the LangGraph routing logic.

### Add New Tools

Add domain-specific tool functions under:

```text
customer_support_chat/app/services/tools/
```

Then expose the tools to the appropriate assistant and update the graph if additional routing or confirmation behavior is required.

### Change Retrieval Behavior

Vector database behavior is implemented under:

```text
customer_support_chat/app/services/vectordb/
```

and:

```text
vectorizer/app/vectordb/
```

These modules can be customized for collection naming, content formatting, chunking, indexing, and search behavior.

### Change Embedding Configuration

Embedding generation is implemented in:

```text
vectorizer/app/embeddings/embedding_generator.py
```

Update this module when changing the embedding model or embedding API behavior.

---

## Security

This project processes travel, booking, and potentially user-specific information. Credentials and application data should be handled carefully.

### Never commit

- OpenAI API keys
- LangSmith API keys
- Production database credentials
- Private customer data
- Production configuration files containing secrets
- Generated private vector database storage
- Other authentication tokens

### Recommended practices

- Keep `.env` files outside version control.
- Use environment variables or a secure secrets manager.
- Use separate development and production credentials.
- Restrict access to Qdrant in production.
- Review tool permissions before enabling sensitive actions.
- Keep human confirmation enabled for operations that modify bookings or other important state.
- Avoid logging sensitive user information unnecessarily.

---

## Future Improvements

The current architecture provides several natural areas for expansion:

### More Advanced RAG

Potential retrieval improvements include:

- Adaptive retrieval
- Corrective retrieval when search results are weak
- Self-evaluation of retrieved context
- Better metadata filtering
- Improved reranking

### Better Memory

Add persistent user/session memory so the assistant can maintain useful context across conversations.

### Expanded Tooling

Additional tools could support:

- Booking modification
- Cancellation workflows
- Customer profile lookup
- Travel-policy lookup
- More sophisticated recommendation flows

### Graph Optimization

The LangGraph workflow can be expanded with more precise conditional routing, validation steps, and recovery paths.

### Production Infrastructure

The architecture can be extended with managed vector storage, scalable application hosting, centralized secrets management, monitoring, and automated deployment pipelines.

---

## Documentation

The repository also contains module-level documentation:

```text
customer_support_chat/README.md
vectorizer/README.md
```

These files provide additional implementation details for their respective components.

---

## License

No `LICENSE` file is present in the repository.

Therefore, this README does **not** claim a specific open-source license.

If this project is intended to be distributed publicly under an open-source license, add the appropriate `LICENSE` file and update this section to match the selected license terms.

---

## Summary

This project provides a modular foundation for a travel-focused AI customer-support system.

Its main pipeline is:

```text
Source Data
    ↓
Vectorization
    ↓
OpenAI Embeddings
    ↓
Qdrant
    ↓
User Query
    ↓
Primary Assistant
    ↓
Specialized Assistant
    ↓
Tools + Retrieval
    ↓
Optional Confirmation
    ↓
Final Response
```

The separation between vectorization, retrieval, assistants, tools, and graph orchestration makes the system suitable for experimentation with multi-agent workflows and for extending the customer-support experience with additional domains and capabilities.
