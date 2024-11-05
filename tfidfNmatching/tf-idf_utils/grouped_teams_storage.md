#### 5.4 Match Group Processing and Storage

---
linked:
  - ![batch processing](./redis_streams-cleanup.md)
---

#### 5.4.1 Time-Based Group Accumulation
```python
# After successful team name validation:
matching_group = "2024-11-04 15:00:00"  # Common start time
grouped_by_start_time[matching_group].append(
    (data_key, path, json_obj)  # Store full entry data
)

# From logs - Progressive group building:
"Added to group with start time: 2024-11-04 15:00:00"
"Group now contains 1 entries"      # First bookmaker
"Group now contains 2 entries"      # Second bookmaker
"Group now contains 3 entries"      # Third bookmaker
```

#### 5.4.2 Match Data Organization
```python
# For groups meeting minimum bookmaker requirement (least_count)
if len(entries) >= least_count:
    match_id = generate_match_id()  # Generate UUID
    match_key = f"matched_teams_{stream_name}:{match_id}"
    
    # Initialize storage containers
    match_team_objects = {}  # Per-bookmaker data
    team_names = []         # Team name variations
    match_date = None      # Common match date

    # Process each entry in group
    for data_key, path, json_obj in entries:
        # Extract and store bookmaker data
        bookmaker = json_obj.get("bookmaker")
        match_date = json_obj.get("target_date")
        team_name = list(json_obj.get("teams", {}).keys())[0]
        
        match_team_objects[bookmaker] = json_obj
        team_names.append(team_name)
        logger.debug(f"Added team {team_name} from bookmaker {bookmaker}")
```

#### 5.4.3 Redis Data Structure
```python
# Final match record structure
redis_data = {
    "match_team_objects": match_team_objects,  # Full bookmaker entries
    "teams": ";".join(team_names),            # All team variations
    "created": get_current_date(),            # Creation timestamp
    "arbitrage": 0,                           # Initial arbitrage values
    "arb_updated": 0,
    "bookie_updated": 0,
    "match_date": match_date,                 # Common match date
    "start_time": start_time,                 # Common start time
    "arbitrage_opportunities": []             # Future calculations
}
```

#### 5.4.4 Redis Storage Operations
```python
# Store match data
redis_db.json().set(
    match_key,        # "matched_teams_xtr-THREE_WAY_upcoming_stream:{UUID}"
    Path.root_path(), 
    redis_data
)

# Add to processing stream
stream_id = redis_db.xadd(stream_key, {"data_key": match_key})

# Log storage success
logger.info(
    f"Updated match info in Redis - Key: {match_key}, "
    f"Teams: {';'.join(team_names)}, "
    f"Match ID: {match_id}, "
    f"Stream: {stream_key}, "
    f"Start Time: {start_time}"
)
```

From logs:
```
Updated match info in Redis - Key: matched_teams_xtr-THREE_WAY_upcoming_stream:6fb872f7-5bf9-4e41-bb8e-b619a4b47103, 
Teams: PSM Makassar;Persik Kediri;PSM Makassar;Persik Kediri;PSM Makassar;Persik Kediri, 
Match ID: 6fb872f7-5bf9-4e41-bb8e-b619a4b47103, 
Stream: upcoming_three_way-matched_stream, 
Start Time: 2024-11-04 15:00:00
```
