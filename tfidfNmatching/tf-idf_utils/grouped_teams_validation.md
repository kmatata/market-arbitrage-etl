#### 5. Group Validation and Processing Phase

---
linked:
  - ![batch processing](./grouped_teams_storage.md)
---

#### 5.1 Redis Update Function Signature
```python
def update_redis_with_grouped_info(
    redis_db,              # Redis connection
    group,                # List of indices from union-find [49, 103, 237]
    index_mapping,        # Position -> (data_key, path) mapping
    stream_key,           # Output stream identifier
    least_count,          # Minimum bookmakers required
    stream_name,          # Stream name for match key generation
    logger                # Logger instance
):
    """
    Process union-find groups into validated match groups and store in Redis.
    
    Args:
        redis_db: Redis connection instance
        group: List of indices representing potentially matched teams
        index_mapping: Dictionary mapping indices to original data locations
        stream_key: Key for output stream
        least_count: Minimum number of bookmakers required for valid group
        stream_name: Stream identifier for key generation
        logger: Logger instance for operation tracking
    
    Process Flow:
    1. Organize matches by start time
    2. Validate team names and temporal alignment
    3. Create match groups meeting bookmaker threshold
    4. Store validated groups in Redis
    """
    grouped_by_start_time = defaultdict(list)
    logger.info(
        f"Starting Redis update for {len(group)} items in stream: {stream_name}"
    )
```

From logs:
```
Starting Redis update for 3 items in stream: xtr-THREE_WAY_upcoming_stream
Processing group indices: [49, 103, 237]
```

#### 5.2 Time-Based Organization
Each group from union-find undergoes temporal validation:
1. Initial Group Reception:
```python
# Input from group formation:
group = [49, 103, 237]  # Indices representing matched teams
logger.info(f"Starting Redis update for {len(group)} items in stream: {stream_name}")

# First organize by start_time to verify temporal matches
grouped_by_start_time = defaultdict(list)
```

2. Process Each Index in Group:
```python
# For index 49:
data_key, path = index_mapping[49]  # Maps back to Bookiebeta's entry
json_obj = redis_db.json().get(data_key, Path(f"$.[{path}]"))[0]

# Extract key info:
start_time = json_obj.get("start_time")  # "2024-11-04 15:00:00"
current_teams = list(json_obj.get("teams", {}).keys())[0]  # "PSM Makassar;Persik Kediri"
```

3. Time-Based Grouping:
```python
# For first entry (Bookiebeta):
grouped_by_start_time["2024-11-04 15:00:00"].append(
    (data_key, path, json_obj)
)

# When processing next entry (Bookiealpha, index 103):
# First compare times and team names
if compare_team_names(
    current_teams,          # Bookiealpha's teams
    existing_teams,         # Bookiebeta's teams already stored
    start_time,            # Bookiealpha's time
    existing_entry[2].get("start_time")  # Bookiebeta's time
):
    matching_group = existing_start_time
```


#### 5.3 Team Name Comparison Logic
The comparison process involves multiple validation layers to ensure accurate matching:

#### 5.3.1 Time Validation
```python
def is_time_similar(time1, time2, max_diff_minutes=0):
    """
    Verify temporal match alignment.
    
    Example from logs:
    time1: "2024-11-04 15:00:00"
    time2: "2024-11-04 15:00:00"
    """
    dt1 = parse_datetime(time1)
    dt2 = parse_datetime(time2)
    
    time_diff = abs((dt1 - dt2).total_seconds() / 60)
    logger.debug(
        f"Time difference between {time1} and {time2}: {time_diff} minutes"
    )
    return time_diff == max_diff_minutes
```

#### 5.3.2 Team Name Processing
```python
def compare_team_names(teams1, teams2, start_time1, start_time2):
    # First verify times match
    if not is_time_similar(start_time1, start_time2):
        logger.info(f"Times too different: {start_time1} vs {start_time2}")
        return False

    # Split compound team names
    teams1_list = teams1.split(";")  # ["PSM Makassar", "Persik Kediri"]
    teams2_list = teams2.split(";")  # ["PSM Makassar", "Persik Kediri"]
    
    # Sort for consistent comparison
    teams1_list = sorted(teams1_list)
    teams2_list = sorted(teams2_list)

    # Validate team count matches
    if len(teams1_list) != len(teams2_list):
        logger.debug(
            f"Different number of teams: {len(teams1_list)} vs {len(teams2_list)}"
        )
        return False
```

#### 5.3.3 Age Group Handling
```python
def extract_age_group(team_name):
    """
    Extract age category (e.g., U20) from team name if present.
    
    From logs:
    "Extracted age group for team name 'PSM Makassar'"
    """
    match = re.search(r"\bU\d+\b", team_name.upper())
    age_group = match.group(0) if match else None
    logger.debug(
        f"Extracted age group '{age_group}' from team name '{team_name}'"
    )
    return age_group

# Age group comparison in main function
for t1, t2 in zip(teams1_list, teams2_list):
    age_group1 = extract_age_group(t1)
    age_group2 = extract_age_group(t2)
    
    if age_group1 != age_group2:
        logger.debug(
            f"Age groups don't match: '{age_group1}' vs '{age_group2}'"
        )
        continue
```

#### 5.3.4 Name Similarity Computation
```python
    # Remove age groups for base comparison
    base_name1 = re.sub(r"\s*U\d+\s*", "", t1).strip()
    base_name2 = re.sub(r"\s*U\d+\s*", "", t2).strip()

    # Calculate similarity score
    similarity = difflib.SequenceMatcher(
        None, 
        base_name1.lower(), 
        base_name2.lower()
    ).ratio()
    
    logger.debug(
        f"Name similarity between '{base_name1}' and '{base_name2}': {similarity:.4f}"
    )

    # Threshold check (0.68)
    if similarity > 0.68:
        logger.info(
            f"Teams matched: '{t1}' and '{t2}' with similarity {similarity:.4f}"
        )
        return True
```

From logs:
```
Comparing teams - Team1: 'PSM Makassar;Persik Kediri', Team2: 'PSM Makassar;Persik Kediri'
Time difference between 2024-11-04 15:00:00 and 2024-11-04 15:00:00: 0.0 minutes
Name similarity between 'PSM Makassar' and 'PSM Makassar': 1.0000
Teams matched: 'PSM Makassar' and 'PSM Makassar' with similarity 1.0000
```
