
---
linked:
  - ![batch processing](./vectorize.md)
---

#### 2.7 Data Structure Population
```python
documents = []              # Sequential team name storage
document_paths = {}         # Bookmaker -> position mapping
index_mapping = {}         # Position -> source data mapping
global_index = 0           # Position tracker

# For each bookmaker's data:
for data_key, items in all_data.items():
    document_paths[data_key] = []  # Initialize position list
    for key, path in items:
        documents.append(key)      # Add team name to sequence
        document_paths[data_key].append(global_index)  # Track position
        index_mapping[global_index] = (data_key, path) # Map back to source
        global_index += 1
```

```python
# Starting state
global_index = 0   # We start at 0
data_key = 'BookieBeta_xtr-THREE_WAY_upcoming:1730561288-1'
items = [('fc dundee;fc kilmarnock', 0)]

# For first team name:
documents.append('fc dundee;fc kilmarnock')        # Position 0 in documents array

# Critical part - global_index used in two places:
document_paths[data_key].append(global_index)      # Store 0 as position reference for Bookiebeta
index_mapping[global_index] = (data_key, 0)        # Store reverse lookup from 0 to Bookiebeta data

# This creates bidirectional mapping:
# 1. document_paths: Bookiebeta -> [0]     (Find all positions for a bookmaker)
# 2. index_mapping: 0 -> (Bookiebeta, 0)   (Find which bookmaker owns this position)

# After global_index increment:
global_index = 1  # Ready for next entry, maintaining unique position tracking
```

This dual storage of global_index creates a two-way reference system:
- Through document_paths we can find all positions a bookmaker owns
- Through index_mapping we can find which bookmaker owns any position

From logs:
```
Prepared 1920 documents for TF-IDF analysis
```

Total documents (1920) represents sum of all team keys:
- Bookiebeta: 563 teams
- Bookiealpha: 693 teams
- Bookiegamma: 664 teams

#### 2.8 Position Tracking Progression
As more teams are processed, the mapping expands:

```python
# After processing complete Bookiebeta dataset (563 teams):
document_paths = {
    'BookieBeta_xtr-THREE_WAY_upcoming:1730561288-1': [0, 1, 2, ..., 562]
}
index_mapping = {
    0: ('BookieBeta_xtr-THREE_WAY_upcoming:1730561288-1', 0),
    1: ('BookieBeta_xtr-THREE_WAY_upcoming:1730561288-1', 1),
    # ... up to 562
}

# Starting Bookiealpha data (global_index now at 563):
document_paths['BookieAlpha_xtr-THREE_WAY_upcoming:1730561305-1'] = [563, 564, ..., 1255]
index_mapping[563] = ('BookieAlpha_xtr-THREE_WAY_upcoming:1730561305-1', 0)
# ... and so on
```

#### 2.9 Reference System Utility
This bidirectional mapping enables:
1. Forward Lookup: Find all teams from a specific bookmaker
   ```python
   # Get all Bookiebeta team positions:
   BookieBeta_positions = document_paths['BookieBeta_xtr-THREE_WAY_upcoming:1730561288-1']
   # Returns: [0, 1, 2, ..., 562]
   ```

2. Reverse Lookup: Find team source from position
   ```python
   # Find which bookmaker owns position 563:
   bookmaker, internal_index = index_mapping[563]
   # Returns: ('BookieAlpha_xtr-THREE_WAY_upcoming:1730561305-1', 0)
   ```

From logs:
```
Processing data key: BookieBeta_xtr-THREE_WAY_upcoming:1730561288-1
...Added 563 keys for data key          # Positions 0-562

Processing data key: BookieAlpha_xtr-THREE_WAY_upcoming:1730561305-1
...Added 693 keys for data key          # Positions 563-1255

Processing data key: BookieGamma_xtr-THREE_WAY_upcoming:1730561300-1
...Added 664 keys for data key          # Positions 1256-1919
```
