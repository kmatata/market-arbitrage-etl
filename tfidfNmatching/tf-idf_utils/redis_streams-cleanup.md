#### 6. Stream Management and Cleanup Phase

#### 6.1 Message Processing Completion
After successful group processing and storage:
```python
# Acknowledge processed messages from batch
for msg_id, _, _ in batch:
    redis_db.xack(stream_name, group_name, msg_id)
    # Example msg_id: '1727462021354-0'
```

#### 6.2 Stream Size Management
```python
def trim_stream(r_db, stream_key, max_len=100):
    """
    Maintain stream at manageable size.
    
    Args:
        r_db: Redis connection
        stream_key: Stream identifier
        max_len: Maximum entries to retain (default: 100)
    """
    try:
        current_length = r_db.xlen(stream_key)
        if current_length > max_len:
            # Get oldest entry information
            oldest_entry = r_db.xrange(stream_key, count=1)
            
            # Perform trim operation
            num_trimmed = r_db.xtrim(
                stream_key, 
                maxlen=max_len, 
                approximate=False
            )
            
            if oldest_entry:
                oldest_id, oldest_data = oldest_entry[0]
                logger.info(
                    f"Trimmed {num_trimmed} entries from stream {stream_key}."
                )
                logger.info(
                    f"Oldest entry removed: ID={oldest_id}, Data={oldest_data}"
                )
```

#### 6.3 Error Handling and Recovery
```python
try:
    # Process group validation and storage
    process_batch(...)
except ResponseError as e:
    logger.critical(f"Redis response error: {e}")
    raise e
except Exception as e:
    logger.critical(f"Unexpected error in processing: {e}")
    raise e
finally:
    # Ensure stream maintenance
    trim_stream(redis_db, stream_name, 100)
```

#### 6.4 Processing Completion States
Successful completion logs:
```
Processing group with start time 2024-11-04 15:00:00, containing 3 entries
Updated match info in Redis - Key: matched_teams_xtr-THREE_WAY_upcoming_stream:...
Successfully processed batch of 3 messages
```

Failed processing logs:
```
Error processing group item: {error_message}
Redis error: {error_details}
```

#### 6.5 Resource Cleanup
- Stream entries older than retention period removed
- Processed messages acknowledged
- Invalid or incomplete groups discarded
- Memory released from temporary processing structures