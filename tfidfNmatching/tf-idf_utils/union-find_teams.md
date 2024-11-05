#### 4. Team Group Formation Phase

---
linked:
  - ![batch processing](./grouped_teams_validation.md)
---

The union operation is where the magic happens

#### 4.1 Union-Find Structure Initialization
```python
# Initialize for all 1920 team positions
parent = list(range(global_index))  # [0, 1, 2, ..., 1919]
rank = [0] * global_index           # [0, 0, 0, ..., 0]

# Initial state for each team:
# - parent[i] = i (each team in its own group)
# - rank[i] = 0 (all trees start with height 0)
```

#### 4.2 Processing Similarity Matches
For each high-similarity pair:
```python
# When processing high similarity matches:
for doc1, doc2, score in formatted_results:
    # For our example match:
    # doc1 = 8 (Bookiebeta team)
    # doc2 = 1080 (Bookiealpha team) 
    # score = 1.0000

    union(parent, rank, doc1, doc2)  # This merges their groups
```

Inside the union operation:
```python
def union(parent, rank, x, y):
    # First find the roots of both teams
    xroot = find(parent, x)    # find(parent, 8)
    yroot = find(parent, y)    # find(parent, 1080)

    # If already in same group, do nothing
    if xroot != yroot:
        # Use rank to keep tree balanced
        if rank[xroot] < rank[yroot]:
            parent[xroot] = yroot    # Smaller tree points to larger
        elif rank[xroot] > rank[yroot]:
            parent[yroot] = xroot
        else:
            parent[yroot] = xroot    # Equal ranks, arbitrarily choose
            rank[xroot] += 1         # Increase height of resulting tree
```

The result:
```python
# After union:
parent[8] = 8        # Root team
parent[1080] = 8     # Bookiealpha team now points to Bookiebeta team
rank[8] = 1         # Tree height increased
rank[1080] = 0      # Leaf node stays at 0
```

This creates an efficient tree structure where:
1. Teams that are matches point to same root
2. find() operation can quickly locate group root
3. rank balancing keeps trees shallow for efficiency


#### 4.3 Tree Formation Example
```python
# Starting state for parent/rank arrays (showing relevant indices):
parent = [0,    1,    2,    8,    ..., 1080,   ..., 1919]  # 1920 total positions
         [0,    1,    2,    8,    ..., 1080,   ..., 1919]  # Each points to self initially
rank =   [0,    0,    0,    0,    ..., 0,      ..., 0]     # All trees start at height 0

# For our match from formatted_results:
# BookieBeta_xtr-THREE_WAY_upcoming:1730561288-1,8 <-> BookieAlpha_xtr-THREE_WAY_upcoming:1730561305-1,1080 
# with similarity 1.0000

# Union called with:
union(parent, rank, 8, 1080)  # Connect Bookiebeta team[8] with Bookiealpha team[1080]

# Inside union:
xroot = find(parent, 8)     # find root of Bookiebeta team
yroot = find(parent, 1080)  # find root of Bookiealpha team

# Initially:
xroot = 8      # Bookiebeta team is its own root
yroot = 1080   # Bookiealpha team is its own root
```


a visualization on how the merging happens in the union operation:

```python
# Initial state before merge:
# parent array:
parent[8] = 8        # Bookiebeta team points to itself (root)
parent[1080] = 1080  # Bookiealpha team points to itself (root)

# rank array:
rank[8] = 0         # Both trees have height 0
rank[1080] = 0

# Inside union, after finding roots (xroot=8, yroot=1080):
if rank[xroot] < rank[yroot]:              # Compare tree heights
    parent[xroot] = yroot                  # Smaller points to larger
elif rank[xroot] > rank[yroot]:
    parent[yroot] = xroot
else:                                      # Equal ranks (our case)
    parent[yroot] = xroot                  # Make Bookiebeta team the root
    rank[xroot] += 1                       # Increase Bookiebeta's tree height
```

After union, our arrays look like:
```python
# parent array updated:
parent[8] = 8        # Bookiebeta team remains root
parent[1080] = 8     # Bookiealpha team now points to Bookiebeta team

# rank array updated:
rank[8] = 1         # Root's height increased
rank[1080] = 0      # Leaf node stays at 0

# Visual tree structure:
#       8 (rank=1)
#       |
#      1080 (rank=0)
```

```python
# When Bookiegamma(1500) matches with Bookiealpha(1080):
union(parent, rank, 1080, 1500)

# First, find the roots:
xroot = find(parent, 1080) # Returns 8 (follows parent[1080] to find root)
yroot = find(parent, 1500) # Returns 1500 (its own root)

# Current state:
parent[8] = 8        # Root of existing tree
parent[1080] = 8     # Points to root from earlier union
parent[1500] = 1500  # Own root initially
rank[8] = 1         # Height from earlier union
rank[1500] = 0      # Starting height

# In union operation:
if rank[xroot] < rank[yroot]:     # 1 < 0? No
    parent[xroot] = yroot         
elif rank[xroot] > rank[yroot]:    # 1 > 0? Yes
    parent[yroot] = xroot         # Make 1500 point to root 8
else:
    parent[yroot] = xroot
    rank[xroot] += 1              # Only increases if ranks were equal

# After union:
parent[1500] = 8    # Points to existing root
rank[8] = 1         # Stays 1 because ranks weren't equal
rank[1500] = 0      # Stays 0 as leaf

# Final tree:
#       8 (rank=1)
#      /  \
#   1080  1500 (both rank=0)
```

Ithe root's rank only increases when merging trees of equal rank. Here, since rank[8]=1 was greater than rank[1500]=0, the rank didn't increase...

#### 4.4 Final Group Extraction
```python

# After all similarity matches processed, we have parent array like:
parent = [0, 1, 2, 8, ..., 1080, ..., 1500, ...]
#                8     -> points to self (root)
#                1080  -> points to 8
#                1500  -> points to 8

# Group formation:
groups = defaultdict(list)
for index in range(global_index):  # Loop through all 1920 indices
    root = find(parent, index)     # Find root for each index
    groups[root].append(index)     # Group by common root
```

This creates structure like:
```python
groups = {
    8: [8, 1080, 1500],    # All teams that matched (Bookiebeta, Bookiealpha, Bookiegamma)
    12: [12, 890, 1600],   # Another group of matched teams
    # ... other groups
}
```

From logs (different instance ex):
```
Starting Redis update for 3 items in stream: xtr-THREE_WAY_upcoming_stream
Processing group indices: [49, 103, 237]

# For first team group:
Processing index 49 - Data key: BookieBeta_xtr-THREE_WAY_upcoming:1730561292-2, Path: 49
Extracted teams from current entry: PSM Makassar;Persik Kediri

Processing index 103 - Data key: BookieAlpha_xtr-THREE_WAY_upcoming:1730561299-2, Path: 15
Extracted teams from current entry: PSM Makassar;Persik Kediri

Processing index 237 - Data key: BookieGamma_xtr-THREE_WAY_upcoming:1730561296-2, Path: 71
Extracted teams from current entry: PSM Makassar;Persik Kediri
```
