# Requests Utility Analysis

- used by: ![etl](../extract_n_load/BookieAlpha),![etl](../extract_n_load/BookieGamma),![etl](../extract_n_load/BookieBeta),![etl](../extract_n_load/BookieDelta)

## Core Function: `make_request`

### Function Signature

```python
async def make_request(
    session,
    url,
    method="GET",
    payload=None,
    headers={},
    ssl=False,
    max_retries=8,
    timeout_duration=30,
    **params
)
```

## Key Components

### 1. Proxy Management

```python
proxies = [
    os.getenv("DYNAMIC_ALWAYS"),
    os.getenv("DYNAMIC_THREE1"),
    os.getenv("DYNAMIC_THREE2"),
    os.getenv("DYNAMIC_PROXY"),
    os.getenv("DYNAMIC_PROXY2"),
]
```

- Maintains a list of 5 rotating proxies
- Proxies are loaded from environment variables
- Rotation happens on each retry attempt

### 2. Request Configuration

```python
timeout = aiohttp.ClientTimeout(total=timeout_duration)
kwargs = {
    "headers": headers,
    "proxy": proxy,
    "params": params,
    "ssl": ssl,
    "timeout": timeout,
}
```

- Default timeout of 30 seconds
- Configurable headers, SSL, and parameters
- Optional payload for non-GET requests

### 3. Retry Mechanism

#### Retry Logic

```python
attempt = 0
while attempt < max_retries:
    proxy = proxies[attempt % len(proxies)]  # Proxy rotation
```

- Maximum of 8 retry attempts
- Rotates through proxies using modulo operation
- Implements exponential backoff

#### Backoff Strategy

```python
await asyncio.sleep(1 * (2 ** (attempt // len(proxies))))
```

- Base delay: 1 second
- Exponential increase after cycling through all proxies
- Example progression:
  1. First 5 attempts: 1 second delay
  2. Next 5 attempts: 2 seconds delay
  3. Next 5 attempts: 4 seconds delay
  4. And so on...

## Error Handling Flow

### 1. HTTP Errors

```python
except aiohttp.ClientError as e:
    if response.status in retry_behaviors:
        print(f"{retry_behaviors[response.status]['message']}, retrying...")
```

- Handles specific HTTP status codes
- Uses predefined retry behaviors
- Logs specific error messages

### 2. Timeout Handling

```python
except asyncio.TimeoutError:
    print("Timeout reached, retrying...")
    await asyncio.sleep(1)
```

- Catches request timeouts
- Implements 1-second delay before retry
- Preserves retry count

### 3. General Exceptions

```python
except Exception as e:
    print(f"Unhandled exception: {e}")
    await asyncio.sleep(0.1)
```

- Catches all other exceptions
- Implements minimal delay
- Continues retry cycle

## Request Flow Diagram

![rfd](../embed/request%20flow.png)

## Usage Examples

### Basic GET Request

```python
async with ClientSession() as session:
    data = await make_request(session, "https://api.instance.com")
```

### POST Request with Payload

```python
async with ClientSession() as session:
    data = await make_request(
        session,
        "https://api.example.com",
        method="POST",
        payload={"key": "value"}
    )
```

### Request with Custom Parameters

```python
async with ClientSession() as session:
    data = await make_request(
        session,
        "https://api.example.com",
        headers={"Authorization": "Bearer token"},
        timeout_duration=60,
        custom_param="value"
    )
```

## Performance Considerations

### 1. Resource Management

- Uses async/await for non-blocking I/O
- Implements session reuse through aiohttp.ClientSession
- Closes connections properly through context managers

### 2. Memory Efficiency

- No response data buffering
- Immediate JSON parsing
- Clean proxy rotation without memory accumulation

### 3. Network Optimization

- Proxy rotation reduces IP-based rate limiting
- Exponential backoff prevents server overload
- Connection pooling through session reuse

## Error Recovery Strategy

1. **First Level**: Retry with different proxy
2. **Second Level**: Increase wait time
3. **Third Level**: Implement exponential backoff
4. **Final Level**: Raise exception after max retries

## Best Practices Implemented

1. **Resilience**:

   - Multiple proxy support
   - Comprehensive error handling
   - Gradual backoff strategy

2. **Maintainability**:

   - Clear function parameters
   - Documented error paths
   - Configurable timeouts and retries

3. **Monitoring**:
   - Request attempt logging
   - Error message clarity
   - Proxy rotation tracking
