# Blog Agentic

Blog Agentic is a small FastAPI and LangGraph project that generates blog content from a topic. It uses a Groq-hosted LLM through LangChain, runs the blog workflow as a LangGraph state graph, and exposes the result through a `/blogs` API endpoint.

## Features

- Generate an SEO-friendly blog title from a topic.
- Generate detailed blog content for the same topic.
- Use LangGraph to organize the generation flow into clear nodes.
- Use FastAPI to expose the workflow as an HTTP API.
- Support local environment variables through `.env`.
- Include LangGraph Studio configuration through `langgraph.json`.

## Project Structure

```text
bloggeneration/
|-- app.py
|-- main.py
|-- langgraph.json
|-- pyproject.toml
|-- requirements.txt
|-- request.json
|-- README.md
`-- src/
    |-- graphs/
    |   `-- graph_builder.py
    |-- llms/
    |   `-- groqllm.py
    |-- nodes/
    |   `-- blog_node.py
    `-- states/
        `-- blogstate.py
```

## How It Works

The application accepts a blog topic and passes it into a LangGraph workflow.

The graph currently has two main nodes:

1. `title_creation`
   - Reads the topic from the graph state.
   - Sends a prompt to the LLM.
   - Creates a creative, SEO-friendly blog title.

2. `content_generation`
   - Reads the topic and generated title from the graph state.
   - Sends another prompt to the LLM.
   - Creates detailed Markdown blog content.

The final response contains a `blog` object with the generated title and content.

## Tech Stack

- Python 3.13+
- FastAPI
- Uvicorn
- LangChain
- LangGraph
- LangChain Groq
- python-dotenv
- Groq LLM API
- LangSmith / LangGraph Studio support

## Requirements

Before running the project, make sure you have:

- Python 3.13 or newer installed.
- A Groq API key.
- Optional: a LangSmith API key if you want tracing or LangGraph Studio support.

## Environment Variables

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key_here
LANGCHAIN_API_KEY=your_langsmith_or_langchain_api_key_here
```

Do not commit your `.env` file to Git. This project ignores `.env` files through `.gitignore`.

If you want to show other developers which variables are required, create a safe template file named `.env.example`:

```env
GROQ_API_KEY=
LANGCHAIN_API_KEY=
```

## Installation

Clone the project and move into the folder:

```bash
git clone <your-repository-url>
cd bloggeneration
```

Create and activate a virtual environment.

On Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

On macOS or Linux:

```bash
python -m venv .venv
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

If you are using `uv`, you can also install from `pyproject.toml`:

```bash
uv sync
```

## Run the API

Start the FastAPI server:

```bash
python app.py
```

The API will run at:

```text
http://localhost:8000
```

You can also start it directly with Uvicorn:

```bash
uvicorn app:app --host 0.0.0.0 --port 8000 --reload
```

## API Usage

### Generate a Blog

Endpoint:

```http
POST /blogs
```

Request body:

```json
{
  "topic": "Agentic AI"
}
```

Example using `curl`:

```bash
curl -X POST "http://localhost:8000/blogs" \
  -H "Content-Type: application/json" \
  -d "{\"topic\":\"Agentic AI\"}"
```

Example response:

```json
{
  "data": {
    "topic": "Agentic AI",
    "blog": {
      "title": "Generated blog title",
      "content": "Generated blog content in Markdown"
    }
  }
}
```

You can also test the API from FastAPI's interactive docs:

```text
http://localhost:8000/docs
```

## LangGraph Studio

This project includes a `langgraph.json` file:

```json
{
  "dependencies": ["."],
  "graphs": {
    "blog_generator_agent": "./src/graphs/graph_builder.py:graph"
  },
  "env": "./.env"
}
```

This exposes the compiled graph as `blog_generator_agent` for LangGraph tools or LangGraph Studio.

If LangGraph CLI is installed, you can run:

```bash
langgraph dev
```

## Important Files

### `app.py`

Defines the FastAPI application and the `/blogs` endpoint. It loads environment variables, creates the Groq LLM, builds the LangGraph workflow, invokes the graph, and returns the generated blog data.

### `src/llms/groqllm.py`

Contains the `GroqLLM` class. This class loads the `GROQ_API_KEY` from `.env` and creates a `ChatGroq` model instance using:

```text
llama-3.1-8b-instant
```

### `src/graphs/graph_builder.py`

Contains the `GraphBuilder` class. It builds the LangGraph workflow by adding nodes and edges:

```text
START -> title_creation -> content_generation -> END
```

### `src/nodes/blog_node.py`

Contains the blog generation logic. It has node functions for:

- Creating a blog title.
- Creating full blog content.

### `src/states/blogstate.py`

Defines the state structure used by the graph. The state includes:

- `topic`
- `blog`
- `current_language`

## Example Workflow

1. User sends a topic to `/blogs`.
2. FastAPI reads the request body.
3. `GroqLLM` creates the LLM object.
4. `GraphBuilder` creates the LangGraph workflow.
5. The graph runs `title_creation`.
6. The graph runs `content_generation`.
7. FastAPI returns the generated blog title and content.

## Development Notes

- Keep `.env` private because it contains API keys.
- Use `.env.example` for public documentation of required environment variables.
- The generated blog content is returned as Markdown text.
- The project currently supports topic-based blog generation.
- The graph can be extended with more nodes, such as outline generation, SEO keyword generation, human review, content editing, or language translation.

## Troubleshooting

### `GROQ_API_KEY` is missing

Make sure your `.env` file exists in the project root and contains:

```env
GROQ_API_KEY=your_groq_api_key_here
```

### API returns an error from the model

Check that:

- Your Groq API key is valid.
- Your account has access to the selected model.
- Your internet connection is working.

### FastAPI server does not start

Check that dependencies are installed:

```bash
pip install -r requirements.txt
```

Then run:

```bash
python app.py
```

## Future Improvements

- Add validation for empty topics.
- Add better error responses for missing API keys.
- Add tests for the graph and API endpoint.
- Add support for tone, audience, language, and word count.
- Add streaming responses for long blog content.
- Add an `.env.example` file.
- Remove debug prints before production deployment.

## License

This project is licensed under the MIT License. See the `LICENSE` file for the SPDX license reference.
