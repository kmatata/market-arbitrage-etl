# Market Arbitrage ETL System

> **Note**: This documentation uses placeholder names (BookieAlpha, BookieBeta, BookieGamma, BookieDelta) to represent actual bookmakers. These names are used for research and educational purposes while maintaining competitive information privacy.

## Overview
A real-time system that monitors multiple sports betting markets to identify pricing inefficiencies and arbitrage opportunities. The system extracts odds from multiple bookmakers (represented by placeholder names), correlates identical matches across platforms using advanced text matching, and analyzes price discrepancies.

## Key Features
- **Real-time Market Monitoring**: Concurrent extraction from multiple bookmakers
- **Intelligent Match Correlation**: TF-IDF-based text analysis for team name matching
- **Market Analysis**: Price difference detection across bookmakers
- **Stream Processing**: Redis-based distributed message handling
- **Containerized Architecture**: Docker-based microservices deployment

## Technology Stack
- **Core**: Python 3.10, Redis Stack (Streams, JSON)
- **Text Analysis**: TF-IDF vectorization, NLTK
- **Infrastructure**: Docker, Redis Pub/Sub
- **Processing**: Async I/O, Concurrent Processing
- **Data Storage**: Redis JSON for structured data

## Use Cases
- Track real-time odds movements across bookmakers
- Identify pricing discrepancies and arbitrage opportunities
- Monitor market response times and odds convergence
- Analyze competitive pricing patterns


---

## Table of Contents:

### 1. [Containerization](/containerize/)

- [extractors Docker Configuration](/containerize/obs_dockerfile.md)

### 2. [Extract and Load](/extract_n_load/)

- [BookieAlpha Extractor](/extract_n_load/BookieAlpha.md)
- [BookieBeta Extractor](/extract_n_load/BookieBeta.md)
- [BookieDelta Extractor](/extract_n_load/BookieDelta.md)
- [BookieGamma Extractor](/extract_n_load/BookieGamma.md)

#### [Data Transformation](/transform/)

- [BookieAlpha Transform](/transform/BookieAlpha%20transformation.md)
- [BookieBeta Transform](/transform/BookieBeta%20transformation.md)
- [BookieDelta Transform](/transform/BookieDelta%20transformation.md)
- [BookieGamma Transform](/transform/BookieGamma%20tranformation.md)

#### [Utilities](/utilities/)

- [Common Utilities](/utilities/obs_common%20utilities.md)
- [Request Utilities](/utilities/obs_requests%20utility.md)

### 3. [System Orchestration](/launcher/)

- [Container Launch System](/launcher/obs_launch_ent-container.md)

### 4. [Data Schemas](/schemas/)

- [Extractor Schemas](/schemas/extractor_schemas.md)

### 5. [TF-IDF & Matching](/tfidfNmatching/)

#### Container

- [TF-IDF Dockerfile](/tfidfNmatching/container/tfidf_dockerfile.md)

#### Launcher

- [TF-IDF Container Launch](/tfidfNmatching/launcher/launch_tfidf_container.md)

#### Core Components

- [Main System](/tfidfNmatching/main.md)
- [Match Entrypoint](/tfidfNmatching/match_entrypoint.md)

#### TF-IDF Utilities

- [Data Structure Population](/tfidfNmatching/tf-idf_utils/data-Struct_population.md)
- [Grouped Teams Storage](/tfidfNmatching/tf-idf_utils/grouped_teams_storage.md)
- [Teams Validation](/tfidfNmatching/tf-idf_utils/grouped_teams_validation.md)
- [Batch Processing](/tfidfNmatching/tf-idf_utils/process_batch.md)
- [Stream Cleanup](/tfidfNmatching/tf-idf_utils/redis_streams-cleanup.md)
- [Union-Find Teams](/tfidfNmatching/tf-idf_utils/union-find_teams.md)
- [Vectorization](/tfidfNmatching/tf-idf_utils/vectorize.md)

#### Tokenization

- [Load Tokenized Stop Words](/tfidfNmatching/tokenization/load_tokenized_stop-words.md)
- [Stop Words and Tokenization](/tfidfNmatching/tokenization/stop%20words%20and%20tokenization%20sys.md)

---


## System Architecture

### 1. Market Data Extraction Service
- Real-time data harvesting through rate-limited API connections
- Handles three key market types:
  - Match Result (1X2)
  - Both Teams to Score (BTTS)
  - Double Chance
- Automated request management with proxy rotation
- Stream-based data publication to Redis

### 2. Team Name Correlation Engine
- TF-IDF-based text analysis for team name matching
- Handles variations in team names across bookmakers
- Examples:
  ```
  "Manchester United" ↔ "Man United" ↔ "Man Utd"
  "Real Madrid" ↔ "R. Madrid" ↔ "Real Mad"
  ```
- Time-windowed batch processing (40-second buckets)
- Union-find algorithm for efficient team grouping

### 3. Data Management System
```
Extraction → Team Matching → Market Analysis
     ↓            ↓               ↓
  Raw Data → Correlated Data → Price Analysis
     ↓            ↓               ↓
Redis Streams → JSON Store → Market Insights
```

## Quick Start
```bash
# Start Redis Stack
docker run -d --name redis-stack --networt xqwer-net redis/redis-stack:latest

# Launch extraction containers
python launcher/start_extractor_containers.py xtr live

# Launch matching engine
python tfidfNmatching/launcher/launch_tfidf_container.py
```

## Performance Considerations
- Handles ~2000 matches per processing window
- 40-second aggregation windows for optimal matching
- Concurrent processing with rate limiting
- Memory-efficient stream processing
- Real-time data synchronization
