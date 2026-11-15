# Movie GraphRAG

A JavaScript GraphRAG project that turns a movie PDF into a searchable knowledge graph and vector index. It combines:

- **Neo4j** for structured movie facts and relationships
- **Pinecone** for semantic similarity and recommendations
- **Google Gemini** for entity extraction, query classification, embeddings, and answer generation

## How It Works

### Indexing

The indexing pipeline reads `data/movies.pdf` and:

1. Extracts structured movie entities with Gemini.
2. Stores movies, people, genres, themes, and awards in Neo4j.
3. Parses and embeds movie text with Gemini.
4. Stores the embeddings and metadata in Pinecone.

### Querying

The interactive query pipeline:

1. Resolves names and other entities against Neo4j.
2. Classifies the question as a graph or similarity query.
3. Routes factual questions to Neo4j.
4. Routes recommendation and similarity questions through Pinecone and Neo4j.
5. Uses Gemini to produce the final answer.

## Prerequisites

- Node.js 18 or newer
- A Neo4j database, such as Neo4j Aura
- A Pinecone account and index
- A Google AI Studio API key with Gemini access
- The source PDF at `data/movies.pdf`

## Installation

```bash
npm install
```

Create a `.env` file in the project root:

```env
NEO4J_URI=neo4j+s://your-instance.databases.neo4j.io
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=your-neo4j-password
NEO4J_DATABASE=neo4j

PINECONE_API_KEY=your-pinecone-api-key
PINECONE_INDEX_NAME=your-pinecone-index

GEMINI_API_KEY=your-gemini-api-key
```

The Pinecone index must be configured for **3072-dimensional** vectors because the project uses `gemini-embedding-001`.

## Commands

Test all service connections:

```bash
npm test
```

Build the Neo4j graph and Pinecone vector store from `data/movies.pdf`:

```bash
npm run index
```

Start the interactive query CLI:

```bash
npm run query
```

Type a question at the prompt and enter `exit` to stop. Example questions:

- `Who directed Inception?`
- `Which movies did Leonardo DiCaprio act in?`
- `Recommend movies similar to Inception`
- `Which sci-fi movies won awards?`

## Project Structure

| File                      | Purpose                                                          |
| ------------------------- | ---------------------------------------------------------------- |
| `1_testConnection.js`     | Verifies Neo4j, Pinecone, Gemini, and embedding connections      |
| `2_config.js`             | Creates and exports shared service clients and embedding helpers |
| `3_pdfParser.js`          | Extracts and chunks text from the source PDF                     |
| `4_entityExtractor.js`    | Extracts structured movie entities with Gemini                   |
| `5_graphBuilder.js`       | Writes movie entities and relationships to Neo4j                 |
| `6_vectorStore.js`        | Embeds movie text and upserts vectors to Pinecone                |
| `7_runIndexing.js`        | Runs the complete indexing pipeline                              |
| `8_cypherTemplates.js`    | Provides safe Cypher query templates                             |
| `9_entityResolver.js`     | Resolves query entities against the Neo4j graph                  |
| `10_queryClassifier.js`   | Classifies questions for graph or similarity handling            |
| `11_graphHandler.js`      | Handles factual and relationship queries with Neo4j              |
| `12_similarityHandler.js` | Handles similarity and recommendation queries                    |
| `13_runQuery.js`          | Runs the interactive query CLI                                   |
| `data/movies.pdf`         | Source movie dataset                                             |

## Data Model

Neo4j stores nodes such as `Movie`, `Director`, `Actor`, `Genre`, `Theme`, and `Award`, connected by relationships including:

- `DIRECTED`
- `ACTED_IN`
- `BELONGS_TO`
- `EXPLORES`
- `WON`

Pinecone stores movie embeddings along with metadata such as title, year, director, actors, genres, themes, and the source text.

## Notes

- Run `npm test` before indexing if this is the first setup.
- Re-running the indexer is intended to be safe because the graph builder uses `MERGE` for graph entities.
- Keep `.env` private. It is excluded from version control by `.gitignore`.
- See `Notes.md` for the detailed GraphRAG explanation and implementation walkthrough.
