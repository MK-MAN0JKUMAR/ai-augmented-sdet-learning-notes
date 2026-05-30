# Phase 3 — Complete Study Guide
## RAG Testing Framework 

> **Prerequisites:** You must have completed Phase 1 and Phase 2.
>
> **How to use this:**
> - Do NOT skip pitfalls — they reveal production issues.
> - This is your main portfolio piece. Take time. Do it right.

---

# INTRODUCTION — Phase 3 Overview

Phase 3 is different from Phase 1 and Phase 2.

**Phase 1–2:** Learning frameworks and concepts  
**Phase 3:** Building your main portfolio project

By end of Phase 3, you will have:
- A working RAG system on GitHub
- Professional documentation
- Evaluation metrics showing system quality
- A project you can show in interviews

This project is what interviewers will ask you about. Make it count.

---

# PART A — RAG FUNDAMENTALS

---

## A1. What is RAG (Conceptual Overview)

**Search in Copilot:** "what is RAG retrieval augmented generation explained"
**Search in Copilot:** "RAG vs pure LLM difference architecture"

### What is RAG:

RAG = Retrieval-Augmented Generation

It combines:
1. **Retrieval** — Find relevant documents from a database
2. **Augmentation** — Include those documents in the prompt
3. **Generation** — LLM generates response using documents

### RAG vs Pure LLM:

```
Pure LLM:
Question → LLM → Answer (based on training data only)
Problem: Can hallucinate, outdated knowledge

RAG:
Question → Retrieve documents → Insert in prompt → LLM → Answer
Advantage: Grounded in your data, current, verifiable
```

### Example RAG:

```
User asks: "What is insurance premium?"
Retriever: Finds documents about insurance from your database
Documents: "Insurance premium is the amount paid for coverage..."
Prompt: "Using this document: [document], answer: What is insurance premium?"
LLM: "Insurance premium is the amount paid for coverage..."
Result: Grounded, verifiable answer
```

### Why RAG for Phase 3:

- **Production reality:** Most AI systems use RAG, not pure LLM
- **Testability:** You can evaluate retrieval AND generation separately
- **Portfolio value:** RAG projects impress interviewers
- **Financial domain:** Insurance/finance heavily use RAG
- **Testing angle:** You can test both components

---

## A2. RAG Pipeline Architecture

**Search in Copilot:** "RAG pipeline architecture components flow"
**Search in Copilot:** "indexing retrieval generation RAG system"

### RAG has 3 stages:

### Stage 1: Indexing (Offline, do once)
```
Raw documents
  ↓
Split into chunks
  ↓
Convert to embeddings (vectors)
  ↓
Store in vector database
```

### Stage 2: Retrieval (Happens per query)
```
User question
  ↓
Convert to embedding
  ↓
Search vector database
  ↓
Get top-k most similar documents
```

### Stage 3: Generation (Happens per query)
```
User question + Retrieved documents
  ↓
Format as prompt
  ↓
Send to LLM
  ↓
LLM generates response
```

### Complete flow:

```
INDEXING (Phase 1: Setup)
Documents → Chunks → Embeddings → Vector DB

RUNTIME (Phase 2: Per question)
Question → Embedding → Search → Retrieve docs → Format prompt → LLM → Response

EVALUATION (Phase 3: Test quality)
Measure retrieval quality (did we get right docs?)
Measure generation quality (is response grounded in docs?)
```

### Design decisions you'll make:

- How to split documents (chunk size)?
- Which embedding model to use?
- Which vector database (FAISS)?
- How many documents to retrieve (top-k)?
- How to format prompt for LLM?
- How to evaluate?

Each decision has trade-offs.

---

## A3. Evaluation in RAG (Different from Phase 2)

**Search in Copilot:** "RAG evaluation retrieval vs generation metrics"
**Search in Copilot:** "how to evaluate RAG system quality"

### Key difference from Phase 2:

Phase 2: Evaluate LLM response quality alone  
Phase 3: Evaluate BOTH retrieval AND generation

### Retrieval Evaluation:

```
Did the retriever get the right documents?

Metrics:
- Retrieval accuracy: Were documents relevant?
- Recall: Did we retrieve all needed docs?
- Precision: Were all retrieved docs useful?
```

### Generation Evaluation:

```
Given the retrieved documents, is the response good?

Metrics (from Phase 2):
- Faithfulness: Is response grounded in docs?
- Answer relevance: Does response answer question?
```

### Example:

```
Question: "What is life insurance?"
Retrieved docs: [Insurance guide, Premiums explained, Coverage types]

Retrieval evaluation:
- Were these docs relevant? YES
- Did we get all needed docs? Maybe (could have more)
- Precision score: 0.85 (mostly useful)

Generation evaluation:
- Is response grounded in docs? YES (0.9 faithfulness)
- Is response relevant to question? YES (0.95 relevance)
```

### Why both matter:

```
Good retrieval + Good generation = Excellent RAG
Good retrieval + Bad generation = Wasted documents
Bad retrieval + Good generation = Hallucination
Bad retrieval + Bad generation = Useless
```

---

# PART B — VECTOR DATABASES AND EMBEDDINGS

---

## B1. What are Embeddings

**Search in Copilot:** "what are embeddings vectors in NLP"
**Search in Copilot:** "how embeddings work semantic similarity"

### Simple definition:

An **embedding** is a numerical representation of text.

### Example:

```
Text: "Insurance is a contract"
Embedding: [0.2, -0.5, 0.8, 0.1, -0.3, ...]  (array of numbers)
```

### Why embeddings matter:

```
Regular text comparison:
"Insurance" == "Insurance" ✓
"Insurance" == "insurance" ✗ (case sensitive)

Embedding comparison:
Embedding("Insurance") ≈ Embedding("insurance") ✓ (similar)
Embedding("Insurance") ≈ Embedding("Coverage") ✓ (related)
```

### Semantic similarity:

```
Embedding space is semantic:
"Insurance" and "Coverage" → Similar embeddings (related concepts)
"Insurance" and "Dog" → Very different embeddings (unrelated)
```

This is how search works in RAG.

### Embedding dimensions:

```
Small embedding (128 dimensions): Fast, less accurate
Medium embedding (384 dimensions): Balanced
Large embedding (1536 dimensions): Slower, more accurate

For Phase 3: Use 384-1536 dimension embeddings
```

---

## B2. FAISS — Vector Database

**Search in Copilot:** "what is FAISS vector database installation"
**Search in Copilot:** "FAISS similarity search how it works"

### What is FAISS:

FAISS = Facebook AI Similarity Search

It's a vector database that:
- Stores embeddings
- Searches quickly
- Finds similar vectors

### Why FAISS for Phase 3:

```
✓ Free (open source)
✓ Fast (optimized for search)
✓ Local (no internet needed)
✓ Simple (easy to use)
✗ Not distributed (single machine)
✗ Not persistent (loses data on restart)
```

For your portfolio, FAISS is perfect.

### How FAISS works:

```
1. Store embeddings in FAISS index
2. Query with question embedding
3. FAISS returns top-k most similar vectors
4. You get the documents those vectors represent
```

### Installation:

```bash
pip install faiss-cpu
# or if you have GPU:
pip install faiss-gpu
```

### Basic usage:

```python
import faiss
import numpy as np

# Create index for 768-dimensional embeddings
embedding_dim = 768
index = faiss.IndexFlatL2(embedding_dim)

# Add embeddings (numpy array, shape: [num_docs, 768])
embeddings = np.random.random((100, 768))
index.add(embeddings.astype('float32'))

# Search (query is 1 embedding)
query_embedding = np.random.random((1, 768))
k = 5  # Get top 5 most similar
distances, indices = index.search(query_embedding.astype('float32'), k)

print(f"Found {k} similar documents")
print(f"Indices: {indices[0]}")  # Document indices
print(f"Distances: {distances[0]}")  # Similarity scores
```

### ⚠️ Pitfall — Embedding dimension mismatch:
```python
# WRONG
embedding_dim = 768
index = faiss.IndexFlatL2(embedding_dim)
embeddings = np.random.random((100, 384))  # Mismatch!
index.add(embeddings)  # ❌ Error: wrong dimension

# CORRECT
embedding_dim = 768
embeddings = np.random.random((100, 768))  # Match!
index.add(embeddings.astype('float32'))  # ✓
```

### ⚠️ Pitfall — Data type:
```python
# WRONG
index.add(embeddings)  # Python float

# CORRECT
index.add(embeddings.astype('float32'))  # numpy float32
```

---

## B3. Embedding Models

**Search in Copilot:** "embedding models sentence transformers hugging face"
**Search in Copilot:** "best open source embedding models 2024"

### Free embedding models:

| Model | Dimensions | Speed | Quality | Use Case |
|-------|-----------|-------|---------|----------|
| all-MiniLM-L6-v2 | 384 | Fast | Good | General, fast |
| all-mpnet-base-v2 | 768 | Medium | Better | Balanced |
| multilingual-e5-large | 1024 | Slow | Best | Multi-language |

### Installation:

```bash
pip install sentence-transformers
```

### Basic usage:

```python
from sentence_transformers import SentenceTransformer

# Load model
model = SentenceTransformer('all-MiniLM-L6-v2')

# Encode documents
documents = [
    "Insurance is a contract for protection",
    "Life insurance covers death benefits",
    "Health insurance covers medical costs"
]
embeddings = model.encode(documents)

# Encode query
query = "What is life insurance?"
query_embedding = model.encode(query)

# Query returns 1 embedding, documents return multiple
print(f"Query shape: {query_embedding.shape}")  # (384,)
print(f"Documents shape: {embeddings.shape}")  # (3, 384)
```

### For Phase 3:

Use `all-MiniLM-L6-v2` or `all-mpnet-base-v2`:
- Fast enough for learning
- Good quality
- 384-768 dimensions (good for FAISS)

### ⚠️ Pitfall — First load is slow:
```python
# First run: Download model (few minutes)
model = SentenceTransformer('all-MiniLM-L6-v2')

# Subsequent runs: Use cached model (fast)
model = SentenceTransformer('all-MiniLM-L6-v2')
```

---

# PART C — LANGCHAIN RAG IMPLEMENTATION

---

## C1. LangChain Overview

**Search in Copilot:** "what is LangChain Python library"
**Search in Copilot:** "LangChain RAG implementation tutorial"

### What is LangChain:

LangChain is a framework that:
- Simplifies LLM application building
- Provides RAG helpers
- Handles embeddings, retrieval, generation

### Installation:

```bash
pip install langchain langchain-community langchain-groq
```

### Basic RAG with LangChain:

```python
from langchain.vectorstores import FAISS
from langchain_groq import ChatGroq
from langchain.chains import RetrievalQA

# 1. Create vector store
vectorstore = FAISS.from_documents(documents, embeddings)

# 2. Create LLM
llm = ChatGroq(model="llama-3.1-8b-instant")

# 3. Create RAG chain
qa = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=vectorstore.as_retriever()
)

# 4. Ask question
response = qa.run("What is insurance?")
```

LangChain handles all the plumbing.

---

## C2. Document Processing

**Search in Copilot:** "document loading text splitting langchain"
**Search in Copilot:** "chunk size document splitting best practices"

### Step 1: Load documents

```python
from langchain.document_loaders import TextLoader

# Load from text file
loader = TextLoader("insurance_guide.txt")
documents = loader.load()

# Load from multiple files
from langchain.document_loaders import DirectoryLoader

loader = DirectoryLoader("./documents/", glob="*.txt")
documents = loader.load()
```

### Step 2: Split documents

```python
from langchain.text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,        # Size of each chunk
    chunk_overlap=100,      # Overlap between chunks
    separators=["\n\n", "\n", ". ", " "]  # Split by these
)

chunks = splitter.split_documents(documents)
print(f"Created {len(chunks)} chunks")
```

### Chunk size consideration:

```
Small chunks (200-300 tokens):
✓ More precise retrieval
✗ May split important concepts

Medium chunks (500-1000 tokens):
✓ Good balance
✓ Most common

Large chunks (2000+ tokens):
✓ Keep context together
✗ May retrieve too much
```

### ⚠️ Pitfall — Chunk overlap:

```python
# Without overlap
Chunk 1: [words 1-100]
Chunk 2: [words 101-200]
Problem: Concepts split across chunks, search misses them

# With overlap
Chunk 1: [words 1-100]
Chunk 2: [words 51-150]  # 50 word overlap
Better: Concepts are complete in chunks
Cost: More chunks to store and search
```

### For Phase 3:

```python
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
    separators=["\n\n", "\n", ".", " "]
)
```

---

## C3. Creating Vector Store

**Search in Copilot:** "langchain FAISS vector store creation"

### Complete example:

```python
from langchain.document_loaders import TextLoader
from langchain.text_splitters import RecursiveCharacterTextSplitter
from langchain.embeddings import HuggingFaceEmbeddings
from langchain.vectorstores import FAISS

# 1. Load documents
loader = TextLoader("insurance_data.txt")
documents = loader.load()
print(f"Loaded {len(documents)} documents")

# 2. Split documents
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50
)
chunks = splitter.split_documents(documents)
print(f"Created {len(chunks)} chunks")

# 3. Create embeddings
embeddings = HuggingFaceEmbeddings(
    model_name="all-MiniLM-L6-v2"
)
print("Embedding model loaded")

# 4. Create vector store
vectorstore = FAISS.from_documents(chunks, embeddings)
print("Vector store created")

# 5. Save for later use
vectorstore.save_local("vectorstore")
print("Vector store saved")
```

### Load saved vector store:

```python
from langchain.vectorstores import FAISS
from langchain.embeddings import HuggingFaceEmbeddings

embeddings = HuggingFaceEmbeddings(
    model_name="all-MiniLM-L6-v2"
)
vectorstore = FAISS.load_local("vectorstore", embeddings)
```

### ⚠️ Pitfall — Embedding model mismatch:

```python
# Created with:
embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
vectorstore = FAISS.from_documents(chunks, embeddings)

# Later, loading with different model:
embeddings = HuggingFaceEmbeddings(model_name="all-mpnet-base-v2")
vectorstore = FAISS.load_local("vectorstore", embeddings)
# ❌ Error: Dimension mismatch

# Correct: Use same model
embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
vectorstore = FAISS.load_local("vectorstore", embeddings)
```

---

## C4. Semantic Search and Retrieval

**Search in Copilot:** "semantic search vector similarity retrieval"

### How retrieval works:

```python
from langchain.vectorstores import FAISS

vectorstore = FAISS.load_local("vectorstore", embeddings)

# Retrieve documents similar to query
query = "What is life insurance?"
retrieved_docs = vectorstore.similarity_search(query, k=5)

print(f"Retrieved {len(retrieved_docs)} documents:")
for i, doc in enumerate(retrieved_docs):
    print(f"\n{i+1}. {doc.page_content[:100]}...")
```

### Similarity scoring:

```python
# Get documents with similarity scores
results = vectorstore.similarity_search_with_scores(
    "What is insurance?",
    k=5
)

for doc, score in results:
    print(f"Score: {score:.2f}")
    print(f"Content: {doc.page_content[:50]}...")
```

Score meaning:
```
0.0 = Identical
0.5 = Similar
1.0 = Different
```

### ⚠️ Pitfall — Insufficient retrieval:

```python
# What if no relevant documents exist?
query = "Tell me about quantum computing"
docs = vectorstore.similarity_search(query, k=5)

# You'll still get 5 documents, even if none are relevant!
# Solution: Check relevance scores or do manual filtering
```

---

# PART D — RAG RETRIEVAL EVALUATION

---

## D1. Evaluating Retrieval Quality

**Search in Copilot:** "retrieval evaluation metrics RAG systems"

### Key retrieval metrics (RAGAS):

From Phase 2, you know:
- **Context Recall:** Did we retrieve all needed documents?
- **Context Precision:** Were retrieved documents relevant?

### Practical retrieval evaluation:

```python
from ragas.metrics import context_recall, context_precision
from langchain.vectorstores import FAISS

vectorstore = FAISS.load_local("vectorstore", embeddings)

# Test question and its ground truth answer
question = "What is life insurance?"
ground_truth = """Life insurance is a contract where an insurer 
promises to pay a death benefit to beneficiaries when the insured 
person dies."""

# Retrieve documents
retrieved_docs = vectorstore.similarity_search(question, k=5)
contexts = [doc.page_content for doc in retrieved_docs]

# Evaluate retrieval
recall_score = context_recall.score(
    question=question,
    ground_truth=ground_truth,
    contexts=contexts
)

precision_score = context_precision.score(
    question=question,
    ground_truth=ground_truth,
    contexts=contexts
)

print(f"Recall: {recall_score:.2f}")
print(f"Precision: {precision_score:.2f}")
```

### Trade-off: Recall vs Precision

```
High Recall, Low Precision:
- Retrieve many documents
- Include relevant ones (good)
- Include irrelevant ones (bad)
- Cost: More tokens sent to LLM

Low Recall, High Precision:
- Retrieve few documents
- All are relevant (good)
- May miss needed info (bad)
- Cost: Cheaper but might hallucinate

Balanced:
- Retrieve top-k, evaluate both
- Adjust k to balance
```

### Optimizing retrieval:

```python
# Test different k values
for k in [3, 5, 10, 15]:
    docs = vectorstore.similarity_search(question, k=k)
    contexts = [doc.page_content for doc in docs]
    
    recall = context_recall.score(
        question=question,
        ground_truth=ground_truth,
        contexts=contexts
    )
    precision = context_precision.score(
        question=question,
        ground_truth=ground_truth,
        contexts=contexts
    )
    
    print(f"k={k}: Recall={recall:.2f}, Precision={precision:.2f}")

# Pick k that balances both
```

---

## D2. Testing Retriever Edge Cases

**Search in Copilot:** "RAG retrieval failures edge cases"

### Common retrieval problems:

```
1. No relevant documents exist
   → Retriever returns irrelevant docs
   → LLM hallucinates
   
2. Query is out-of-domain
   → Retriever can't find matches
   → LLM has no context
   
3. Query is too vague
   → Retriever returns diverse docs
   → LLM confused which to use
   
4. Documents are poorly chunked
   → Chunks split important concepts
   → Retriever can't find complete info
```

### Test cases you should have:

```python
# Test 1: Valid question with relevant docs
question = "What is life insurance?"
# Expected: Retrieve relevant docs, high scores

# Test 2: Question outside knowledge base
question = "What is the stock price of Apple?"
# Expected: Low relevance scores, no good matches

# Test 3: Ambiguous question
question = "Coverage"
# Expected: Multiple different types retrieved

# Test 4: Multi-part question
question = "What is insurance and how does it work?"
# Expected: Docs covering both aspects (may need more context)
```

---

# PART E — RAG GENERATION EVALUATION

---

## E1. Evaluating Generation in RAG Context

**Search in Copilot:** "evaluating LLM generation with retrieved context"

### From Phase 2, you know:

- **Faithfulness:** Is response grounded in source?
- **Answer Relevance:** Does response answer question?

### In RAG, evaluation changes:

```
Phase 2 evaluation: Is response good?
Phase 3 evaluation: Is response good given THESE documents?

Important: Evaluation must know the retrieved documents!
```

### Example:

```python
from ragas.metrics import faithfulness, answer_relevancy
from langchain.vectorstores import FAISS

question = "What is life insurance?"
retrieved_docs = vectorstore.similarity_search(question, k=5)
contexts = [doc.page_content for doc in retrieved_docs]

# Get LLM response
llm_response = "Life insurance is a contract..."

# Evaluate WITH context
faithfulness_score = faithfulness.score(
    question=question,
    answer=llm_response,
    contexts=contexts  # Important!
)

relevance_score = answer_relevancy.score(
    question=question,
    answer=llm_response
)

print(f"Faithfulness: {faithfulness_score:.2f}")
print(f"Relevance: {relevance_score:.2f}")
```

### Critical difference:

```
Without context (Phase 2):
"Is this response good?" → Depends on LLM training

With context (Phase 3):
"Is this response grounded in THESE documents?" → More specific

In RAG, second question matters more!
```

---

## E2. Measuring Hallucination in RAG

**Search in Copilot:** "hallucination detection RAG systems"

### Hallucination in RAG context:

```
Question: "What is life insurance?"
Retrieved docs: [Only about health insurance]
LLM response: "Life insurance covers death benefits"

Is this hallucination?
- Not in retrieved docs (hallucination in RAG sense)
- But factually correct (not hallucination in general sense)

For RAG testing: It IS a problem because it's not grounded in retrieval!
```

### Testing hallucination in RAG:

```python
from deepeval.metrics import Hallucination
from deepeval.test_cases import LLMTestCase

question = "What is life insurance?"
answer = "Life insurance covers death benefits and provides..."
contexts = [doc.page_content for doc in retrieved_docs]

test_case = LLMTestCase(
    input=question,
    actual_output=answer,
    retrieval_context=contexts  # Include retrieved context
)

metric = Hallucination()
metric.measure(test_case)

print(f"Hallucination score: {metric.score}")
print(f"Reason: {metric.reason}")
```

### ⚠️ Pitfall — Hallucination in poor retrieval:

```
If retriever returns irrelevant docs:
- LLM has no good context
- LLM likely hallucinates
- Hallucination detector flags it
- But ROOT CAUSE is retrieval!

Solution:
Always check both retrieval AND generation quality.
Bad retrieval will cause bad generation.
```

---

# PART F — END-TO-END RAG SYSTEM

---

## F1. Complete RAG Pipeline

**Search in Copilot:** "complete RAG pipeline implementation langchain"

### Full workflow:

```python
from langchain.document_loaders import TextLoader
from langchain.text_splitters import RecursiveCharacterTextSplitter
from langchain.embeddings import HuggingFaceEmbeddings
from langchain.vectorstores import FAISS
from langchain_groq import ChatGroq
from langchain.chains import RetrievalQA

# 1. INDEXING (do once, offline)
print("Loading documents...")
loader = TextLoader("insurance_data.txt")
documents = loader.load()

print("Splitting into chunks...")
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50
)
chunks = splitter.split_documents(documents)

print("Creating embeddings...")
embeddings = HuggingFaceEmbeddings(
    model_name="all-MiniLM-L6-v2"
)

print("Building vector store...")
vectorstore = FAISS.from_documents(chunks, embeddings)
vectorstore.save_local("vectorstore")

# 2. RUNTIME (per query)
print("\nLoading vector store...")
vectorstore = FAISS.load_local("vectorstore", embeddings)

print("Setting up LLM...")
llm = ChatGroq(model="llama-3.1-8b-instant")

print("Creating RAG chain...")
qa = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=vectorstore.as_retriever(search_kwargs={"k": 5})
)

# 3. ANSWER QUESTIONS
question = "What is life insurance?"
print(f"\nQuestion: {question}")
answer = qa.run(question)
print(f"Answer: {answer}")
```

### Chain type explanation:

```
"stuff" — Stuff all docs into one prompt
  ✓ Simple, works for small retrieval sets
  ✗ Fails if documents too long (exceeds context window)

"refine" — Refine answer by processing docs one by one
  ✓ Handles large document sets
  ✗ Slower, more API calls

"map_reduce" — Process each doc, then summarize
  ✓ Handles very large sets
  ✗ Most expensive

For Phase 3: Use "stuff" (simplest, works for portfolios)
```

---

## F2. RAG System with Metadata

**Search in Copilot:** "langchain documents metadata source tracking"

### Including metadata:

```python
from langchain.schema import Document

# Documents with metadata
docs = [
    Document(
        page_content="Life insurance provides death benefits...",
        metadata={"source": "insurance_guide.pdf", "page": 1}
    ),
    Document(
        page_content="Health insurance covers medical costs...",
        metadata={"source": "insurance_guide.pdf", "page": 2}
    )
]

# Create vector store with metadata
vectorstore = FAISS.from_documents(docs, embeddings)

# Retrieve with metadata
docs_with_meta = vectorstore.similarity_search("What is insurance?")
for doc in docs_with_meta:
    print(f"Content: {doc.page_content[:50]}...")
    print(f"Source: {doc.metadata['source']}")
    print(f"Page: {doc.metadata['page']}")
```

### Why metadata matters:

```
✓ Track where information came from
✓ Show sources in evaluation reports
✓ Verify grounding
✓ Debug retrieval issues
```

---

## F3. Streaming Responses (For User Experience)

**Search in Copilot:** "langchain streaming response LLM"

### Streaming (show response as it generates):

```python
qa = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=vectorstore.as_retriever(),
    return_source_documents=True  # Include source docs
)

# Get answer and sources
result = qa({"query": "What is life insurance?"})
answer = result["result"]
sources = result["source_documents"]

print(f"Answer: {answer}\n")
print("Sources:")
for doc in sources:
    print(f"- {doc.metadata.get('source', 'Unknown')}")
```

---

# PART G — PORTFOLIO PROJECT STRUCTURE

---

## G1. RAG Testing Framework Project

**This is your Phase 3 exit deliverable.**

### Project structure:

```
rag-testing-framework/
├── README.md                           # Project overview
├── ARCHITECTURE.md                     # System design
├── requirements.txt                    # Dependencies
│
├── data/
│   ├── insurance_faqs.txt             # Sample data
│   └── test_questions.json            # Test dataset
│
├── src/
│   ├── indexing.py                    # Build vector store
│   ├── retrieval.py                   # Retrieval logic
│   ├── generation.py                  # RAG chain logic
│   ├── evaluation.py                  # Evaluation metrics
│   └── config.py                      # Configuration
│
├── tests/
│   ├── test_retrieval.py              # Retrieval tests
│   ├── test_generation.py             # Generation tests
│   └── test_end_to_end.py             # Full pipeline tests
│
├── notebooks/
│   └── rag_demo.ipynb                 # Interactive demo
│
├── results/
│   ├── retrieval_metrics.json         # Retrieval evaluation
│   ├── generation_metrics.json        # Generation evaluation
│   └── report.html                    # Visual report
│
└── .github/
    └── workflows/
        └── evaluation.yml              # CI/CD pipeline
```

---

## G2. Core Modules

### indexing.py

```python
"""
Build and save vector store from documents
"""
from langchain.document_loaders import TextLoader
from langchain.text_splitters import RecursiveCharacterTextSplitter
from langchain.embeddings import HuggingFaceEmbeddings
from langchain.vectorstores import FAISS

def build_vectorstore(data_path: str, vectorstore_path: str):
    """Build vector store from documents"""
    
    # Load documents
    loader = TextLoader(data_path)
    documents = loader.load()
    
    # Split into chunks
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=500,
        chunk_overlap=50
    )
    chunks = splitter.split_documents(documents)
    
    # Create embeddings
    embeddings = HuggingFaceEmbeddings(
        model_name="all-MiniLM-L6-v2"
    )
    
    # Build and save
    vectorstore = FAISS.from_documents(chunks, embeddings)
    vectorstore.save_local(vectorstore_path)
    
    return len(chunks)

if __name__ == "__main__":
    num_chunks = build_vectorstore(
        "data/insurance_faqs.txt",
        "vectorstore"
    )
    print(f"Built vector store with {num_chunks} chunks")
```

### retrieval.py

```python
"""
Evaluate retrieval quality
"""
from langchain.vectorstores import FAISS
from langchain.embeddings import HuggingFaceEmbeddings
from ragas.metrics import context_recall, context_precision

def evaluate_retrieval(
    vectorstore,
    question: str,
    ground_truth: str,
    k: int = 5
):
    """Evaluate how well documents were retrieved"""
    
    # Retrieve documents
    docs = vectorstore.similarity_search(question, k=k)
    contexts = [doc.page_content for doc in docs]
    
    # Calculate metrics
    recall = context_recall.score(
        question=question,
        ground_truth=ground_truth,
        contexts=contexts
    )
    
    precision = context_precision.score(
        question=question,
        ground_truth=ground_truth,
        contexts=contexts
    )
    
    return {
        "recall": recall,
        "precision": precision,
        "k": k,
        "num_retrieved": len(docs)
    }
```

### generation.py

```python
"""
RAG pipeline and generation evaluation
"""
from langchain_groq import ChatGroq
from langchain.chains import RetrievalQA

def create_rag_chain(vectorstore):
    """Create RAG chain"""
    llm = ChatGroq(model="llama-3.1-8b-instant")
    
    qa = RetrievalQA.from_chain_type(
        llm=llm,
        chain_type="stuff",
        retriever=vectorstore.as_retriever(
            search_kwargs={"k": 5}
        ),
        return_source_documents=True
    )
    
    return qa

def evaluate_generation(
    qa_chain,
    question: str,
    contexts: list
):
    """Evaluate generation quality"""
    
    result = qa_chain({"query": question})
    answer = result["result"]
    
    # Your Phase 2 evaluation here
    # (faithfulness, relevance, hallucination)
    
    return {
        "answer": answer,
        "contexts_used": len(result["source_documents"])
    }
```

### evaluation.py

```python
"""
Complete evaluation metrics
"""
from retrieval import evaluate_retrieval
from generation import evaluate_generation
from ragas.metrics import faithfulness

def evaluate_rag_system(qa_chain, vectorstore, test_cases):
    """End-to-end evaluation"""
    
    results = {
        "retrieval": [],
        "generation": [],
        "end_to_end": []
    }
    
    for test in test_cases:
        question = test["question"]
        ground_truth = test["ground_truth"]
        
        # Retrieval evaluation
        ret_metrics = evaluate_retrieval(
            vectorstore,
            question,
            ground_truth
        )
        results["retrieval"].append(ret_metrics)
        
        # Generation evaluation
        gen_metrics = evaluate_generation(
            qa_chain,
            question,
            []  # Simplified
        )
        results["generation"].append(gen_metrics)
    
    return results
```

---

## G3. GitHub Actions CI/CD

**File:** `.github/workflows/evaluation.yml`

```yaml
name: RAG Evaluation

on: [push, pull_request]

jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
      
      - name: Build vector store
        run: python src/indexing.py
      
      - name: Run evaluations
        run: python src/evaluation.py
      
      - name: Generate report
        run: python src/report.py
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v2
        with:
          name: evaluation-results
          path: results/
```

---

## G4. HTML Report Generation

**File:** `src/report.py`

```python
"""
Generate HTML report of evaluation
"""
import json
from datetime import datetime

def generate_html_report(results, output_path="results/report.html"):
    """Generate visual report"""
    
    html = f"""
    <html>
    <head>
        <title>RAG Testing Report</title>
        <style>
            body {{ font-family: Arial; margin: 20px; }}
            .section {{ background: #f0f0f0; padding: 15px; margin: 10px 0; border-radius: 5px; }}
            .metric {{ display: inline-block; margin: 10px 20px 10px 0; }}
            .good {{ color: green; }}
            .warning {{ color: orange; }}
            .bad {{ color: red; }}
            table {{ border-collapse: collapse; width: 100%; }}
            th, td {{ border: 1px solid #ddd; padding: 8px; text-align: left; }}
            th {{ background: #4CAF50; color: white; }}
        </style>
    </head>
    <body>
        <h1>RAG System Evaluation Report</h1>
        <p>Generated: {datetime.now().isoformat()}</p>
        
        <div class="section">
            <h2>Summary</h2>
            <div class="metric">
                <strong>Retrieval Recall:</strong> 
                <span class="good">{results['avg_recall']:.2f}</span>
            </div>
            <div class="metric">
                <strong>Retrieval Precision:</strong> 
                <span class="good">{results['avg_precision']:.2f}</span>
            </div>
            <div class="metric">
                <strong>Generation Faithfulness:</strong> 
                <span class="good">{results['avg_faithfulness']:.2f}</span>
            </div>
        </div>
        
        <div class="section">
            <h2>Detailed Results</h2>
            <table>
                <tr>
                    <th>Question</th>
                    <th>Recall</th>
                    <th>Precision</th>
                    <th>Faithfulness</th>
                </tr>
    """
    
    for result in results['detailed']:
        html += f"""
        <tr>
            <td>{result['question'][:50]}...</td>
            <td>{result['recall']:.2f}</td>
            <td>{result['precision']:.2f}</td>
            <td>{result['faithfulness']:.2f}</td>
        </tr>
        """
    
    html += """
            </table>
        </div>
    </body>
    </html>
    """
    
    with open(output_path, "w") as f:
        f.write(html)
```

---

# PART H — PHASE 3 EXIT PROJECT

---

## H1. Phase 3 Exit Project Specification

**You must complete this to finish Phase 3.**

### Deliverables:

1. **Working RAG System**
   - Indexes financial/insurance documents
   - Retrieves relevant documents
   - Generates LLM responses
   - All code in GitHub

2. **Evaluation Framework**
   - Measures retrieval quality (recall, precision)
   - Measures generation quality (faithfulness, relevance)
   - Produces JSON results
   - Produces HTML report

3. **Portfolio Documentation**
   - README.md explaining system
   - ARCHITECTURE.md with design decisions
   - Explains trade-offs
   - Links to code

4. **GitHub Repository**
   - Clean structure
   - All code with comments
   - CI/CD pipeline set up
   - Evaluation runs automatically

5. **Demo**
   - Can run RAG locally
   - Can show retrieval
   - Can show generation
   - Can explain evaluation results

### Success criteria:

- [ ] RAG system works end-to-end
- [ ] Retrieval evaluation implemented
- [ ] Generation evaluation implemented
- [ ] HTML report generated
- [ ] GitHub has clean documentation
- [ ] Can demo in 15 minutes
- [ ] Can explain every design decision
- [ ] Can discuss trade-offs
- [ ] Hiring panel impressed

---

## H2. What to Build (Concrete Steps)

### Step 1: Prepare Data
Get insurance/financial FAQ data:
```
Option A: Use public FAQ dataset
Option B: Create synthetic Q&A
Option C: Use sample data (provided in template)

Put in: data/insurance_faqs.txt
```

### Step 2: Build Indexing
```bash
python src/indexing.py
# Creates: vectorstore/ folder
```

### Step 3: Test Retrieval
```bash
python src/test_retrieval.py
# Outputs: retrieval_metrics.json
```

### Step 4: Build RAG Chain
```bash
python src/generation.py
# Tests: RAG chain works
```

### Step 5: Run Evaluations
```bash
python src/evaluation.py
# Outputs: Full evaluation results
```

### Step 6: Generate Report
```bash
python src/report.py
# Creates: results/report.html
```

### Step 7: Push to GitHub
```bash
git add .
git commit -m "Phase 3: RAG Testing Framework"
git push origin main
```

---

## H3. Interview-Ready Answers

After Phase 3, you must be able to answer:

1. **"Walk me through your RAG system"**
   - Answer: Explain indexing → retrieval → generation
   - Show: Architecture diagram
   - Demonstrate: Running the system

2. **"Why did you split documents this way?"**
   - Answer: Chunk size trade-offs
   - Show: Before/after metrics with different chunk sizes
   - Explain: Your choice and why

3. **"How do you evaluate retrieval?"**
   - Answer: Context recall and precision
   - Show: RAGAS metrics from your system
   - Explain: What each means

4. **"What's the biggest challenge in RAG?"**
   - Answer: Balancing retrieval and generation
   - Show: Examples from your project
   - Explain: How you solved it

5. **"What would you do differently?"**
   - Answer: Show awareness of trade-offs
   - Mention: Alternatives you considered
   - Explain: Why you picked your approach

---

# PHASE 3 — TOPIC CHECKLIST

### RAG Concepts (Part A)
- [ ] A1. RAG architecture and flow
- [ ] A2. RAG pipeline components
- [ ] A3. Evaluation in RAG systems

### Vector Databases (Part B)
- [ ] B1. Embeddings and semantic similarity
- [ ] B2. FAISS vector database setup
- [ ] B3. Embedding models and selection
- [ ] B4. Creating and saving vector stores

### LangChain Implementation (Part C)
- [ ] C1. LangChain overview
- [ ] C2. Document loading and processing
- [ ] C3. Creating vector stores
- [ ] C4. Semantic search and retrieval

### Retrieval Evaluation (Part D)
- [ ] D1. Context recall and precision metrics
- [ ] D2. Testing retriever edge cases

### Generation Evaluation (Part E)
- [ ] E1. Evaluating generation with context
- [ ] E2. Hallucination detection in RAG

### End-to-End System (Part F)
- [ ] F1. Complete RAG pipeline
- [ ] F2. RAG with metadata tracking
- [ ] F3. Streaming responses

### Portfolio Project (Part G)
- [ ] G1. Project structure planning
- [ ] G2. Core modules implementation
- [ ] G3. GitHub Actions setup
- [ ] G4. HTML report generation

### Exit Project (Part H)
- [ ] H1. Deliverables complete
- [ ] H2. All steps executed
- [ ] H3. Interview answers prepared

---

# PHASE 3 EXIT CONDITION

You are ready for Phase 4 when you can:

1. ✓ **Demo your RAG system**
   - Run it locally
   - Show retrieval working
   - Show generation working
   - Answer questions

2. ✓ **Explain every component**
   - Why you chose chunk size
   - Why you chose embedding model
   - Why you chose retrieval method
   - Why you chose evaluation metrics

3. ✓ **Discuss trade-offs**
   - Chunk size vs retrieval accuracy
   - Embedding model vs speed
   - k value vs cost/quality
   - Different chain types

4. ✓ **Show measurable results**
   - Retrieval metrics (recall, precision)
   - Generation metrics (faithfulness, relevance)
   - HTML report with visualizations

5. ✓ **Have clean GitHub project**
   - Professional documentation
   - Clear code structure
   - CI/CD working
   - Ready for interviews

---

# COMMON PHASE 3 PITFALLS

| Pitfall | Why | Fix |
|---------|-----|-----|
| Bad data quality | Garbage in, garbage out | Use clean, well-formatted data |
| Chunk size too small | Splits concepts, retrieval fails | Use 500-1000 tokens |
| Chunk size too large | LLM confused, slow | Use 500-1000 tokens |
| Wrong embedding model | Dimension mismatch | Use consistent model |
| Too many documents | FAISS slow, context window exceeded | Filter or use summarization |
| No metadata tracking | Can't verify sources | Include metadata in documents |
| Ignoring retrieval quality | Generation fails silently | Always test retrieval first |
| Not evaluating | Can't improve | Use RAGAS metrics |

---

*Phase 3 Complete Study Guide | Month 4–7 | RAG Testing Framework Portfolio Project*
