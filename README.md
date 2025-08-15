# HeyGen Streaming SDK for Python

An asynchronous Python client for interacting with the HeyGen Streaming API, designed for high-performance and reliability.

## Features

- Full async/await support
- Type hints throughout
- Pydantic models for request/response validation
- Environment-based configuration
- Singleton client instance
- Built-in error handling

## Installation

```bash
# Using pip
pip install heygen-streaming-sdk

# Using poetry
poetry add heygen-streaming-sdk
```

## Configuration

Configuration is handled through environment variables. Create a `.env` file in your project root:

```bash
HEYGEN_API_KEY=your_api_key_here
HEYGEN_BASE_URL=https://api.heygen.com/v1
HEYGEN_TIMEOUT=30
```

Or set them in your environment:

```bash
export HEYGEN_API_KEY=your_api_key_here
export HEYGEN_BASE_URL=https://api.heygen.com/v1
export HEYGEN_TIMEOUT=30
```

## Usage

### Basic Usage

```python
from heygen_streaming import client

async def create_streaming_session():
    # The client is a singleton, so you can import and use it directly
    session = await client.create_session(
        # Your session configuration here
    )
    return session
```

### Using with FastAPI

```python
from fastapi import FastAPI, Depends
from heygen_streaming import client, config

app = FastAPI()

@app.get("/create-session")
async def create_session():
    """Create a new streaming session."""
    try:
        session = await client.create_session(
            # Your session configuration here
        )
        return {"session_id": session.id}
    except Exception as e:
        return {"error": str(e)}, 500
```

## API Reference

### `HeyGenStreamingClient`

The main client class for interacting with the HeyGen Streaming API.

#### Methods

- `create_session(request: NewSessionRequest) -> NewSessionResponse`: Create a new streaming session.
- `start()`: Initialize the HTTP client.
- `close()`: Close the HTTP client.

### Configuration

Configuration is available through the `config` object:

```python
from heygen_streaming.config import config

print(f"Using API base URL: {config.BASE_URL}")
```

## Error Handling

The SDK provides the following exceptions:

- `HeyGenAPIError`: Base exception for all API errors
- `AuthenticationError`: Raised for authentication failures
- `ValidationError`: Raised for request/response validation failures

## Development

### Setup

1. Clone the repository
2. Install dependencies:
   ```bash
   poetry install
   ```
3. Create a `.env` file based on `.env.example`

### Testing

Run the test suite:

```bash
pytest tests/
```

### Linting

```bash
ruff check .
```

### Formatting

```bash
black .
```

## License

MIT
