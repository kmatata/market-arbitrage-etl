#### 3. TF-IDF Processing Phase

---
linked:
  - ![batch processing](./union-find_teams.md)
---

#### 3.1 Vector Creation
The sequential document storage enables consistent vectorization:

```python
# After all documents are loaded:
global_index = 1920  # Total number of team names across all bookmakers

# Initialize union-find data structures
parent = list(range(global_index))  # [0, 1, 2, ..., 1919]
rank = [0] * global_index           # [0, 0, 0, ..., 0] (1920 zeros)

```

The parent and rank arrays are crucial because:
1. parent tracks potential matches (indices that should be grouped)
2. rank maintains balanced trees for efficient grouping operations
3. Both arrays span all 1920 indices, matching our global_index space

```python
# Initialize vectorizer with stop words
vectorizer = TfidfVectorizer(stop_words=tokenized_stop_words)

# Given these team names:
documents = [
    'fc dundee;fc kilmarnock',      # Bookiebeta's version
    'dundee fc;kilmarnock fc',      # Bookiealpha's version
    'dundee fc;kilmarnock fc'       # Bookiegamma' version
]

# And our tokenized_stop_words containing:
tokenized_stop_words = ['fc', 'united', 'city', 'real', ...]

# fit_transform does:
# 1. Tokenizes each team name, removing stop words:
# 'fc dundee;fc kilmarnock' -> ['dundee', 'kilmarnock']
# 'dundee fc;kilmarnock fc' -> ['dundee', 'kilmarnock']

# 2. Creates sparse matrix where:
# - Rows represent team names
# - Columns represent unique terms
# - Values are TF-IDF scores

# Create vectors for all 1920 team names
combined_matrix = vectorizer.fit_transform(documents)
# combined_matrix might look like (in dense form for visualization):
#               'dundee'    'kilmarnock'
# doc[0]:     [0.707,      0.707]        # fc dundee;fc kilmarnock
# doc[1]:     [0.707,      0.707]        # dundee fc;kilmarnock fc
# doc[2]:     [0.707,      0.707]        # dundee fc;kilmarnock fc

# The 0.707 values are normalized TF-IDF scores where:
# - TF (Term Frequency) = 1 (each term appears once)
# - IDF (Inverse Document Frequency) = same for both terms as they appear in all documents
# - Normalized so each row's values square sum to 1 (√0.707² + 0.707² = 1)

# This representation shows why matches can be found - identical team names (after stop word removal) get identical vectors, enabling similarity comparison.

```

#### 3.2 Bookmaker-Specific Matrix Organization
```python
# Initialize per-bookmaker matrices
tfidf_matrices = {}
start = 0

# Split combined matrix by bookmaker using document_paths
for data_key, positions in document_paths.items():
    end = start + len(positions)
    tfidf_matrices[data_key] = combined_matrix[start:end]
    start = end

# Results in:
tfidf_matrices = {
    'BookieBeta_xtr-THREE_WAY_upcoming:1730561288-1': matrix[0:563],      # Bookiebeta vectors
    'BookieAlpha_xtr-THREE_WAY_upcoming:1730561305-1': matrix[563:1256],    # Bookiealpha vectors
    'BookieGamma_xtr-THREE_WAY_upcoming:1730561300-1': matrix[1256:1920]   # Bookiegamma vectors
}
```

#### 3.3 Similarity Computation
```python
# Compare each bookmaker pair

for i in range(len(data_keys)):
    for j in range(i + 1, len(data_keys)):
        matrix1 = tfidf_matrices[data_keys[i]]  # e.g., Bookiebeta's vectors
        matrix2 = tfidf_matrices[data_keys[j]]  # e.g., Bookiealpha's vectors
        
        # Find similarities with threshold
        C = sp_matmul_topn(
            matrix1, 
            matrix2.transpose(), 
            top_n=2000, 
            threshold=0.56
        )
```

What sp_matmul_topn does:
1. Performs sparse matrix multiplication between bookmakers' vectors
2. For each team in matrix1, finds teams in matrix2 with cosine similarity > 0.56
3. Returns only top 2000 highest similarities per team
4. Does this efficiently by:
   - Only computing high-similarity pairs
   - Avoiding dense matrix operations
   - Skipping calculations likely to be below threshold

From logs:
```
Checking for matches
Formatting similarity results
Formatted 390 similarity results

BookieBeta_xtr-THREE_WAY_upcoming:1730561288-1,8 <-> BookieAlpha_xtr-THREE_WAY_upcoming:1730561305-1,1080 with similarity 1.0000
BookieBeta_xtr-THREE_WAY_upcoming:1730561288-1,28 <-> BookieAlpha_xtr-THREE_WAY_upcoming:1730561305-1,636 with similarity 1.0000
```

The efficiency is crucial because we're comparing:
- Bookiebeta (563 teams) × Bookiealpha (693 teams)
- Potentially 390,159 comparisons, but `sp_matmul_topn` only returns promising matches

#### 3.4 Result Interpretation
The bidirectional mapping enables tracing matches:
```python
# For similarity result (8, 1080, 1.0000):
# 1. Position 8 maps to Bookiebeta team via index_mapping[8]
# 2. Position 1080 maps to Bookiealpha team via index_mapping[1080]
# 3. Perfect similarity (1.0000) indicates exact team name match after standardization
```
