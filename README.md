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

## Abstract

A distributed real-time system designed to analyze and correlate sports betting markets across multiple bookmakers. The system comprises three main applications:

1. **Market Data Extraction Service**

   - Real-time data harvesting from multiple bookmakers
   - Automated rate-limiting and request management
   - Stream-based data publication

2. **Market Correlation Engine**

   - TF-IDF-based text analysis for team name matching
   - Time-windowed batch processing(mostly due to the architecture of application, given the market entries need to be temporarily stored; can probably be better implemented)
   - Cross-bookmaker market validation
   - Automated team name reconciliation

3. **Data Management System**
   - Redis-based stream processing
   - Distributed message buffering
   - Containerized deployment management
   - Resource optimization and monitoring

## Motivation

The development of this system was primarily driven by observing competitive market dynamics in the sports betting industry:

### Market Competition Dynamics

- **Competitive Odds Positioning**

  - Bookmakers actively adjust odds to appear more attractive than competitors; inflation of odds on specific markets to drive customer acquisition
  - Temporary promotional odds boosts during peak betting periods

- **Market Coverage Competition**

  - Expansion of available betting markets to differentiate from competitors

- **Customer Acquisition Strategies**
  - Use of enhanced odds as marketing tools
  - Market-specific promotional adjustments

### Business Intelligence Needs

The system addresses these dynamics by:

1. **Real-time Market Monitoring**

   - Tracking odds changes across platforms
   - Identifying promotional patterns

2. **Market Efficiency Assessment**
   - Analyzing price discrepancies
   - Tracking market response times
   - Monitoring odds convergence patterns

This understanding of market dynamics and competitive behavior forms the foundation for the system's design and implementation.

## System Overview

This repository contains technical documentation for a distributed bookie data extraction and processing system. The system is designed to collect, transform, and analyze bookie data from multiple sources in real-time, with advanced text matching capabilities for market reconciliation.

## Key Capabilities

- Real-time data extraction from multiple bookie sources
- Distributed processing using Redis streams
- Advanced TF-IDF-based market matching system
- Time-windowed batch processing
- Containerized deployment architecture
- Scalable data transformation pipeline
- Automated data synchronization
- Real-time market analysis and reconciliation

## Documentation Structure

### Core Components

- **Data Extraction**: Integration with multiple data sources
- **Stream Processing**: Redis-based message handling
- **Market Matching**: TF-IDF-based text analysis system
  - Message buffering and batch formation
  - Temporal windowing (40-second buckets)
  - Cross-bookmaker validation
  - Team name reconciliation
- **Data Transformation**: Standardization and processing pipelines
- **Schema Management**: Redis-based data organization
- **System Orchestration**: Container and process management

### Architecture Diagrams

Located in `/embed/`, providing visual representations of:

- System flows and data pipelines
- Stream processing and batch formation
- TF-IDF matching architecture
- Component interactions
- Deployment architecture

### Technical Documentation

- Extraction processes
- Stream processing flows
- Match analysis pipelines
  - Message buffering mechanics
  - Batch formation criteria
  - TF-IDF processing
  - Team grouping and validation
- Schema definitions
- Utility functions
- Container management

## System Architecture

it operates in three main phases:

1. **Data Extraction & Streaming**

   - Multiple bookmaker data sources
   - Real-time extraction
   - Stream publication

2. **Message Processing & Batch Formation**

   - Time-windowed buffering
   - Cross-bookmaker batch assembly
   - Age-based message management

3. **Market Analysis & Matching**
   - TF-IDF-based text analysis
   - Team name reconciliation
   - Market validation
   - Result storage

![System Architecture](./embed/launcher_sysArch.png)

## Documentation Only

**This repository contains only technical documentation and architectural diagrams. No source code is included.**

## Note

This is a documentation-only repository focused on system architecture and capabilities.(**FOR RESEARCH | EDUCATIONAL PURPOSES**)

- Bookmaker names are obscured

## Key Documentation Sections

### 1. Data Integration & Stream Processing

- Real-time data extraction
- Multiple source support
- Stream-based message handling
- Error handling and retry mechanisms
- Rate limiting and request management

### 2. Market Matching Features

- Time-windowed batch processing
- TF-IDF text analysis
- Cross-bookmaker validation
- Team name reconciliation
- Market correlation

### 3. System Components

- Redis stream management
- TF-IDF processing container
- Message buffer system
- Process monitoring
- Resource optimization

### 4. Architecture Patterns

- Distributed stream processing
- Event-driven architecture
- Time-windowed batching
- Message buffering
- Containerization

## Documentation Organization

### 1. Component Documentation

Each component's documentation includes:

- Architectural overview
- Process flows
- Integration points
- Key features
- Stream handling patterns

### 2. Flow Diagrams

Organized by component:

- Extraction flows
- Stream processing sequences
- Batch formation patterns
- TF-IDF processing flows
- System interactions
- Deployment patterns

### 3. Integration Patterns

Documentation covers:

- Stream-based communication
- Batch processing mechanics
- Error handling strategies
- Resource management
- Cross-component synchronization

## Diagram Categories

### 1. Flow Diagrams

- `*_flow*.png`: System and component flows
- `*_seqFlow.png`: Sequence diagrams
- `*_FD.png`: Functional diagrams
- `obs_* | *_obs`: obscured

### 2. Architecture Diagrams

- System architecture
- Component relationships
- Deployment patterns
- Integration flows

## Documentation Focus Areas

### 1. System Architecture

- Component relationships
- Data flows
- Integration patterns
- Scaling considerations

### 2. Data Processing

- Extraction mechanisms
- Transformation pipelines
- Data validation
- Market analysis

### 3. Deployment Architecture

- Container orchestration
- Resource management
- Process monitoring
- System scaling

## Usage Guidelines

### 1. Navigation

- Use diagram references for visual understanding
- Follow component documentation for detailed information
- Cross-reference integration patterns
- Review transformation flows

### 2. Documentation Reading

- Start with high-level architecture
- Review component interactions
- Understand data flows
- Examine specific implementations

### 3. Architecture Understanding

- Review system diagrams
- Follow data flow patterns
- Understand component relationships
- Study integration approaches

## Summary

This documentation repository provides:

- Comprehensive system architecture overview
- Detailed component documentation
- Visual process flows
- Stream processing patterns
- Match analysis architecture
- Integration patterns
- Deployment strategies
- Scaling considerations

The documentation illustrates the capabilities and architecture of a distributed bookies' market analysis system, focusing on real-time data processing and advanced text matching capabilities, without exposing implementation details.
