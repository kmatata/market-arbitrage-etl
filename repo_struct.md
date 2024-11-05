
market-arbitrage-etl/
├── containerization/              # Container configurations
│   └── obs_dockerfile.md
├── embed/
├── extract_n_load/                   # Source extraction documentation
│   ├── BookieAlpha.md
│   ├── BookieBeta.md
│   ├── BookieDelta.md
│   └── BookieGamma.md
├── launcher/                     # System orchestration
│   └── obs_launch_ent-container.md
├── schemas/                      # Data schema documentation
│   └── extractor_schemas.md
├── tfidfNmatching
│   ├── container
│   │   └── tfidf_dockerfile.md
│   ├── embed/
│   ├── launcher
│   │   └── launch_tfidf_container.md
│   ├── main.md
│   ├── match_entrypoint.md
│   ├── tf-idf_utils
│   │   ├── data-Struct_population.md
│   │   ├── grouped_teams_storage.md
│   │   ├── grouped_teams_validation.md
│   │   ├── process_batch.md
│   │   ├── redis_streams-cleanup.md
│   │   ├── union-find_teams.md
│   │   └── vectorize.md
│   └── tokenization
│       ├── load_tokenized_stop-words.md
│       └── stop words and tokenization sys.md
├── transform/              # Data transformation docs
│   ├── BookieAlpha transformation.md
│   ├── BookieBeta transformation.md
│   ├── BookieDelta transformation.md
│   └── BookieGamma tranformation.md
└── utilities/                   # Shared utilities docs
    ├── obs_common utilities.md
    └── obs_requests utility.md