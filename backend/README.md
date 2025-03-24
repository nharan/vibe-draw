# Claude 3.7 Backend Service

A Python backend service that integrates with Claude 3.7 using FastAPI, Celery, Redis, and Server-Sent Events (SSE).

## Features

- Synchronous and asynchronous prompt processing with Claude 3.7
- Real-time streaming of Claude responses using SSE
- Task queueing and background processing with Celery
- Redis for message broker, result backend, and pub/sub
- Simple and easy-to-use REST API

## Installation

1. Clone the repository
2. Install the dependencies: `pip install -r requirements.txt`
3. Copy `.env.example` to `.env` and update with your configuration

## Environment Variables

```
# API configurations
API_HOST=0.0.0.0
API_PORT=8000

# Redis configurations
REDIS_HOST=localhost
REDIS_PORT=6379

# Celery configurations
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/0

# API keys
ANTHROPIC_API_KEY=your_anthropic_api_key_here
GOOGLE_API_KEY=your_google_api_key_here
CEREBRAS_API_KEY=your_cerebras_api_key_here
TRELLIS_API_KEY=your_trellis_api_key_here
```

## Running the Application

### Start Redis (if not already running)

```bash
redis-server
```

### Start the API server

```bash
python run.py
```

### Start the Celery worker

```bash
celery -A worker worker --loglevel=info
```

## API Endpoints

### 1. Synchronous Claude Prompt

```
POST /api/claude/prompt
```

Sends a prompt to Claude and waits for the response.

**Request Body:**

```json
{
  "prompt": "Tell me a joke about programming",
  "system_prompt": "You are a helpful assistant",
  "max_tokens": 4096,
  "temperature": 0.7
}
```

### 2. Asynchronous Claude Prompt

```
POST /api/claude/prompt/async
```

Sends a prompt to Claude asynchronously and returns a task ID.

**Request Body:** Same as synchronous endpoint

### 3. Get Task Status

```
GET /api/claude/task/{task_id}
```

Returns the status and result of an asynchronous task.

### 4. Start Streaming Claude Response

```
POST /api/claude/queue
```

Starts a streaming response from Claude.

**Request Body:** Same as synchronous endpoint, with optional `task_id`

### 5. Stream Claude Events

```
GET /api/claude/subscribe/{task_id}
```

Streams events from a Claude task using Server-Sent Events (SSE).

### 6. Create Trellis 3D Task

```
POST /api/trellis/task
```

Creates a task in the Trellis API for image-to-3D generation. Uses the same format as the Trellis API create-task endpoint.

**Request Body:**

```json
{
  "model": "Qubico/trellis",
  "task_type": "image-to-3d",
  "input": {
    "image": "base64_encoded_image_or_image_url",
    "seed": 0,
    "ss_sampling_steps": 50,
    "slat_sampling_steps": 50,
    "ss_guidance_strength": 7.5,
    "slat_guidance_strength": 3
  },
  "config": {
    "webhook_config": {
      "endpoint": "optional_webhook_endpoint",
      "secret": "optional_webhook_secret"
    }
  }
}
```

**Response:**
Returns the response from the Trellis API, which includes the job ID.

### 7. Trellis 3D Task Status WebSocket

```
WebSocket: /api/trellis/task/ws/{task_id}
```

Connects to a WebSocket that continuously polls the Trellis API for the status of a 3D generation task. The WebSocket will check the task status every 2 seconds and return the 3D model when it's ready.

**URL Parameters:**
- `task_id`: The ID of the task to poll (obtained from the create task endpoint)

**WebSocket Events:**
- Connection sends status updates in JSON format
- When the task is completed, returns the 3D model data
- Client can send "close" to immediately close the connection

**WebSocket Response Format:**
```json
{
  "status": "processing|completed|failed|error",
  "message": "Status message",
  "data": "URL to 3D model file when completed, otherwise null",
  "full_response": "Complete response from Trellis API when task is completed or failed"
}
```

**Example Usage:**
```javascript
// Connect to the WebSocket
const ws = new WebSocket('ws://localhost:8000/api/trellis/task/ws/your-task-id');

// Handle messages from the server
ws.onmessage = function(event) {
  const data = JSON.parse(event.data);
  
  if (data.status === 'completed') {
    console.log('3D model ready:', data.data);
    ws.close();
  } else if (data.status === 'failed' || data.status === 'error') {
    console.error('Error:', data.message);
    ws.close();
  } else {
    console.log('Status update:', data.message);
  }
};

// Close the connection
ws.onclose = function() {
  console.log('Connection closed');
};

// To manually close the connection
function closeConnection() {
  ws.send('close');
}
```

## Example Usage

### Sending a prompt and waiting for response

```python
import requests

response = requests.post(
    "http://localhost:8000/api/claude/prompt",
    json={
        "prompt": "Explain quantum computing in simple terms",
        "system_prompt": "You are a helpful assistant",
        "max_tokens": 4096,
        "temperature": 0.7
    }
)

print(response.json())
```

### Streaming a response

```javascript
// In browser JavaScript
const eventSource = new EventSource('http://localhost:8000/api/claude/subscribe/your-task-id');

eventSource.addEventListener('start', (event) => {
  console.log('Stream started:', JSON.parse(event.data));
});

eventSource.addEventListener('chunk', (event) => {
  const data = JSON.parse(event.data);
  console.log('Received chunk:', data.chunk);
});

eventSource.addEventListener('complete', (event) => {
  console.log('Stream completed:', JSON.parse(event.data));
  eventSource.close();
});

eventSource.addEventListener('error', (event) => {
  console.error('Stream error:', JSON.parse(event.data));
  eventSource.close();
});
```
