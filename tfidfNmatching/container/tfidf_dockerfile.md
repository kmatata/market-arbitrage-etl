# TF-IDF Match Processing System - Container Runtime Documentation

## Container Environment Configuration

---
linked:
    - ![container entrypoint](../launcher/launch_tfidf_container.md)
---

### 1. Base Image Configuration (`dockerfile.tfidfmatching`)

```dockerfile
FROM python:3.10

# Environment settings
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Security setup
RUN useradd -m -d /home/tfidf -s /bin/bash tfidf
```

Key Features:
- Python 3.10 base image
- No bytecode generation
- Unbuffered output
- Dedicated service user

### 2. Directory Structure
```dockerfile
# Application directories
WORKDIR /main

# Log directory setup
RUN mkdir -p /var/log/analyzeMatch \
    && chown -R tfidf:tfidf /var/log/analyzeMatch
```

Directory Layout:
```
/main/
├── commands/
│   ├── arb_match.py
│   └── load_stop_words.py
├── utils/
│   ├── redis_helper.py
│   └── logger.py
└── main.py
```

### 3. Environment Variables
```python
# Runtime configuration
ENV REDIS_HOST=redis-stack

# Container-specific variables
PREFIX          # 'xtr' or 'lst'
CATEGORY        # 'btts', 'three_way', 'double_chance'
PERIOD          # 'live' or 'upcoming'
```

## Container Initialization Flow

### 1. Pre-Runtime Setup
```python
# Load dependencies
COPY ./requirements.txt /main/requirements.txt
RUN pip install --upgrade pip && pip install -r /main/requirements.txt

# Environment configuration
COPY ./.env main/.env
```

### 2. Application Loading
```dockerfile
# Copy application components
COPY ./main/commands/ /main/commands/
COPY ./utils/ /main/utils/
COPY ./main.py /main/main.py
```

### 3. Runtime Configuration
```dockerfile
USER tfidf
ENV REDIS_HOST=redis-stack
CMD ["python", "/main/main.py"]
```

## Container Security Implementation

### 1. User Context
```dockerfile
# Create non-root user
RUN useradd -m -d /home/tfidf -s /bin/bash tfidf

# Set ownership
RUN chown -R tfidf:tfidf /var/log/analyzeMatch
```

Security Features:
- Non-root execution
- Limited permissions
- Dedicated home directory
- Controlled log access

### 2. Network Configuration
```python
# From launcher logs
container = client.containers.run(
    image_name,
    detach=True,
    name=container_name,
    network="chrome-net",
    **container_options
)
```

Network Features:
- Isolated network
- Inter-container communication
- Redis connectivity
- External access control

## Container Resource Management

### 1. Log Management
```python
# From logger configuration
MAX_LOG_SIZE = 1 * 1024 * 1024  # 1 MB
BACKUP_COUNT = 5
MAX_LOG_AGE_HOURS = 2

# Container log structure
/var/log/analyzeMatch/
├── container_name_YYYY-MM-DD.log
├── container_name_YYYY-MM-DD.log.1
└── container_name_YYYY-MM-DD.log.2
```

### 2. Process Isolation
```python
# Container process hierarchy
tfidf           # Service user
 └── python     # Main process
     └── main.py
```

## Integration Points

### 1. Redis Connection
```python
# From redis_helper.py
def get_redis_connection():
    host = os.getenv("REDIS_HOST")  # "redis-stack"
    port = int(os.getenv("REDIS_PORT"))
    db = int(os.getenv("REDIS_DB"))
```

### 2. Stop Signal Handling
```python
# From launcher monitoring
if check_stop_file():
    logger.info("Stop file detected. Stopping containers...")
    stop_containers(docker_client, active_containers)
```

## Runtime Monitoring

### 1. Log Output
```python
# Container logs show processing state
2024-11-02 15:28:26,456 - commands.arb_match - INFO - 
Processing batch of 3 messages

2024-11-02 15:28:26,456 - commands.arb_match - INFO - 
Fetching data for 3 keys
```

### 2. Health Checks
```bash
# Container status verification
docker ps --format "{{.Names}}: {{.Status}}" | grep tfidf

# Log monitoring
tail -f /var/log/analyzeMatch/container_name_*.log
```

## Best Practices

### 1. Security
- Non-root execution
- Minimal permissions
- Resource isolation
- Secure defaults

### 2. Resource Management
- Log rotation
- Size limits
- Age-based cleanup
- Memory management

### 3. Operations
- Health monitoring
- Error tracking
- Resource cleanup
- State management

## Production Deployment

### 1. Container Launch
```bash
# Build image
docker build -t match/tfidf -f dockerfile.tfidfmatching .

# Runtime validation
docker logs match_tfidf_container
```

### 2. Network Setup
```bash
# Network creation
docker network create chrome-net

# Container attachment
docker network connect chrome-net match_tfidf_container
```

### 3. Monitoring
```bash
# Container health
docker stats match_tfidf_container

# Log inspection
tail -f /var/log/analyzeMatch/*.log
```