# Market Arbitrage ETL System

---

linked:

- ![BookieAlpha](./extract_n_load/BookieAlpha.md)
- ![BookieGamma](./extract_n_load/BookieGamma.md)
- ![BookieBeta](./extract_n_load/BookieBeta.md)
- ![BookieDelta](./extract_n_load/BookieDelta.md)
- ![container init](./launcher/obs_launch_ent-container.md)

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

3. **Market Efficiency Assessment**
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