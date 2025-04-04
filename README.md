# Readability API

A standalone API service built on top of Mozilla's Readability.js library that extracts clean, readable content from web pages.

## Features

- Extract readable content from any URL
- Parse raw HTML content directly
- Clean and format article content
- Extract metadata (title, author, date, etc.)
- RESTful API with token authentication
- Rate limiting support
- Cross-origin support (CORS enabled)

## Installation

### Standard Installation

1. Clone the repository:
```bash
git clone <your-repository-url>
cd readability-api
```

2. Install dependencies:
```bash
npm install
```

3. Create environment configuration:
```bash
cp .env.example .env
```

4. Configure your environment variables in `.env`:
```bash
# API Configuration
API_TOKEN=your-secret-token-here
PORT=3000

# Rate Limiting Configuration
RATE_LIMIT_WINDOW_MS=900000 # 15 minutes in milliseconds
RATE_LIMIT_MAX=100 # Max requests per window

# Axios Configuration
AXIOS_TIMEOUT=10000 # Timeout in milliseconds
```

5. Start the server:
```bash
node index.js
```

The server will start at `http://localhost:3000` (or your configured PORT).

### Docker Installation

1. Using docker-compose (recommended):
```bash
# Configure your .env file first
docker-compose up -d
```

2. Using Docker directly:
```bash
docker run -p 3000:3000 \
  -e API_TOKEN=your-secret-token \
  -e PORT=3000 \
  -e RATE_LIMIT_WINDOW_MS=900000 \
  -e RATE_LIMIT_MAX=100 \
  -e AXIOS_TIMEOUT=10000 \
  -d milandicic/readability-api:latest
```

> **Note**: The Dockerfile uses `npm ci --omit=dev` for deterministic dependency installation. Make sure you have a valid package-lock.json file if you're building the Docker image yourself.

## API Documentation

### Authentication

All API endpoints (except documentation) require Bearer token authentication.

Add the following header to your requests:
```
Authorization: Bearer your-secret-token-here
```

The token should match the `API_TOKEN` environment variable you configured.

### Endpoints

#### API Documentation
Get the API documentation in HTML format.

**Endpoint:** `GET /api/docs`

**Authentication:** Not required

**Response:** HTML documentation

#### 1. Parse URL
Extract readable content from a URL.

**Endpoint:** `POST /api/parse`

**Authentication:** Required

**Request Body:**
```json
{
    "url": "https://example.com/article"
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "title": "Article Title",
        "byline": "Author Name",
        "content": "Article content in HTML format",
        "textContent": "Plain text content",
        "length": 12345,
        "excerpt": "Article excerpt",
        "siteName": "Site Name",
        "lang": "en"
    }
}
```

#### 2. Parse HTML
Parse raw HTML content directly.

**Endpoint:** `POST /api/parse-html`

**Authentication:** Required

**Request Body:**
```json
{
    "html": "<!DOCTYPE html><html><body>Your HTML content here...</body></html>",
    "url": "https://example.com/article" // Optional: helps with relative URLs
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "title": "Article Title",
        "byline": "Author Name",
        "content": "Article content in HTML format",
        "textContent": "Plain text content",
        "length": 12345,
        "excerpt": "Article excerpt",
        "siteName": "Site Name",
        "lang": "en"
    }
}
```

### Error Responses

The API returns structured error responses:

```json
{
    "success": false,
    "error": "Error message"
}
```

Common error status codes:
- 400: Bad Request (invalid input)
- 401: Unauthorized (missing token)
- 403: Forbidden (invalid token)
- 429: Too Many Requests (rate limit exceeded)
- 500: Internal Server Error
- 504: Gateway Timeout

## Usage Examples

### cURL

#### Parse URL:
```bash
curl -X POST \
  -H "Authorization: Bearer your-secret-token-here" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com/article"}' \
  http://localhost:3000/api/parse
```

#### Parse HTML:
```bash
curl -X POST \
  -H "Authorization: Bearer your-secret-token-here" \
  -H "Content-Type: application/json" \
  -d '{"html": "<html><body>Content</body></html>", "url": "https://example.com"}' \
  http://localhost:3000/api/parse-html
```

### JavaScript (Node.js)

```javascript
const axios = require('axios');

// Configure API client
const API_URL = 'http://localhost:3000';
const API_TOKEN = 'your-secret-token-here';

const client = axios.create({
  baseURL: API_URL,
  headers: {
    'Authorization': `Bearer ${API_TOKEN}`,
    'Content-Type': 'application/json'
  }
});

// Parse URL example
async function parseUrl(url) {
  try {
    const response = await client.post('/api/parse', { url });
    return response.data;
  } catch (error) {
    console.error('Error parsing URL:', error.response?.data || error.message);
    throw error;
  }
}

// Parse HTML example
async function parseHtml(html, url) {
  try {
    const response = await client.post('/api/parse-html', { html, url });
    return response.data;
  } catch (error) {
    console.error('Error parsing HTML:', error.response?.data || error.message);
    throw error;
  }
}

// Usage
parseUrl('https://example.com/article')
  .then(result => console.log(result))
  .catch(err => console.error(err));
```

### Python

```python
import requests
import json

API_URL = 'http://localhost:3000'
API_TOKEN = 'your-secret-token-here'

headers = {
    'Authorization': f'Bearer {API_TOKEN}',
    'Content-Type': 'application/json'
}

# Parse URL example
def parse_url(url):
    try:
        response = requests.post(
            f'{API_URL}/api/parse',
            headers=headers,
            json={'url': url}
        )
        response.raise_for_status()
        return response.json()
    except requests.exceptions.RequestException as e:
        print(f"Error parsing URL: {str(e)}")
        if hasattr(e, 'response') and e.response is not None:
            print(e.response.text)
        raise

# Parse HTML example
def parse_html(html, url=None):
    payload = {'html': html}
    if url:
        payload['url'] = url
        
    try:
        response = requests.post(
            f'{API_URL}/api/parse-html',
            headers=headers,
            json=payload
        )
        response.raise_for_status()
        return response.json()
    except requests.exceptions.RequestException as e:
        print(f"Error parsing HTML: {str(e)}")
        if hasattr(e, 'response') and e.response is not None:
            print(e.response.text)
        raise

# Usage
result = parse_url('https://example.com/article')
print(json.dumps(result, indent=2))
```

## Configuration Options

| Environment Variable | Description | Default |
|---------------------|-------------|---------|
| API_TOKEN | Authentication token for API access | Required |
| PORT | Server port number | 3000 |
| RATE_LIMIT_WINDOW_MS | Rate limit time window in milliseconds | 900000 (15 minutes) |
| RATE_LIMIT_MAX | Maximum requests per rate limit window | 100 |
| AXIOS_TIMEOUT | HTTP request timeout in milliseconds | 10000 (10 seconds) |

## Advanced Usage

The API is configured to work seamlessly behind a reverse proxy. The `trust proxy` setting is enabled by default to properly handle client IP addresses for rate limiting.

### Security Headers

The API uses Helmet middleware to automatically set security-related HTTP headers including:
- Content-Security-Policy
- X-XSS-Protection
- X-Frame-Options
- X-Content-Type-Options
- Strict-Transport-Security

These headers help protect against common web vulnerabilities such as XSS, clickjacking, and content type sniffing.

### Graceful Shutdown

The API implements a graceful shutdown mechanism that:
- Properly handles termination signals (SIGTERM, SIGINT)
- Stops accepting new connections while completing in-progress requests
- Performs a clean shutdown with a 10-second timeout for lingering connections
- Logs the shutdown process

This ensures the application behaves reliably in containerized environments (such as Docker) and during manual restarts.

### Handling Large HTML Content

The API accepts HTML content up to 10MB in size. If you need to process larger files, you can modify the `limit` parameter in the `express.json()` middleware configuration.

## Security Features

The API includes several security features to protect against common vulnerabilities:

### SSRF Protection

The API implements robust Server-Side Request Forgery (SSRF) protection:

- All URLs are validated against private/reserved IP ranges
- Only HTTP and HTTPS protocols are allowed
- DNS resolution is checked before making requests
- Blocked IP ranges include:
  - Private networks (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)
  - Loopback addresses (127.0.0.0/8, ::1)
  - Link-local addresses (169.254.0.0/16)
  - AWS/GCP metadata service IPs
  - Other reserved ranges

### DoS Protection

Protection against Denial of Service attacks:

- Rate limiting based on IP address
- 10MB request size limit
- 10-second timeout on HTML parsing
- Resource constraints on JSDOM to prevent resource exhaustion
- Disabled external resource loading during parsing

### Authentication

Strong API token authentication:

- Bearer token required for all API endpoints
- Uses crypto.timingSafeEqual for constant-time token comparison
- Token must be generated with high entropy

### Other Security Measures

- Helmet middleware for security headers
- CORS protection
- Express-validator for input validation
- Structured logging with sensitive data redaction

## Advanced Configuration

The API provides detailed configuration options for various components:

### Structured Logging

The API uses Pino for structured JSON logging:

```bash
# Set log level
LOG_LEVEL=debug # Options: trace, debug, info, warn, error, fatal
```

In development, logs are formatted for readability using pino-pretty. In production, logs are output as JSON for easier integration with log aggregation systems.

### Running Tests

The project includes automated tests to verify security and functionality:

```bash
# Run all tests
npm test

# Run tests with coverage report
npm run test:coverage

# Run tests in watch mode (during development)
npm run test:watch
```

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- [Mozilla's Readability.js](https://github.com/mozilla/readability) - The core library for content extraction
- [JSDOM](https://github.com/jsdom/jsdom) - For HTML parsing and DOM manipulation 