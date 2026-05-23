# Phase 2 — Complete Study Guide
## LLM Evaluation Foundation | Month 2–4

> **Prerequisites:** You must have completed Phase 1. You can call LLM APIs, understand tokens, temperature, and non-determinism.
>
> **How to use this:**
> - In office (Copilot): Search each topic heading. Read theory only. Take notes.
> - At home (laptop): Run code examples. Understand outputs. Modify and experiment.
> - Do NOT skip pitfalls — they reveal why evaluation frameworks fail in production.
> - You know Phase 1. This builds directly on it.

---

# INTRODUCTION — Why Phase 2 Matters (Read This First)

In Phase 1, you learned that `assertEquals(actual, expected)` fails for LLM outputs. Phase 2 solves that problem.

**The core challenge:**
You have a question, an LLM response, and a source document. How do you know if the response is good?

**Three possible answers:**
1. Manual scoring — slow, expensive, not scalable
2. Rule-based validation — brittle, misses nuance
3. **Automated evaluation using metrics** — what Phase 2 teaches you

Phase 2 is the bridge between "I know LLMs are non-deterministic" and "I know how to test them at scale."

By end of Phase 2, you will have a working test suite that evaluates LLM outputs programmatically. That test suite is your portfolio piece for interviews.

---

# PART A — EVALUATION FRAMEWORKS: CONCEPTS

---

## A1. What Is LLM Evaluation (Conceptual Overview)

**Search in Copilot:** "LLM evaluation metrics what are they why needed"
**Search in Copilot:** "how to measure LLM response quality automatically"

### What to know:

Evaluation metrics are functions that score LLM outputs on specific dimensions.

**Example:**
```
Input question: "What is the capital of India?"
LLM response: "New Delhi is the capital of India."
Source document: "New Delhi (official name: Naya Delhi) is India's capital and union territory..."

Metric 1 (Relevance): Does the response answer the question? → Score 10/10
Metric 2 (Faithfulness): Is the response grounded in the source? → Score 10/10
Metric 3 (Hallucination): Does it claim facts not in source? → Score 0 (no hallucinations)
```

### Why this matters for you:
- In production, you cannot manually check 1,000 responses
- You need automated scoring
- These scores become your test assertions: `assert faithfulness_score > 0.8`

### The two categories of evaluation:

| Type | How | Example | Speed | Cost |
|---|---|---|---|---|
| **Offline** | Use reference answer or source doc | Compare response to source | Fast | Low |
| **Online** | Use another LLM call to judge | LLM-as-judge pattern | Medium | Medium |

Both are useful. You will use both.

---

## A2. DeepEval vs RAGAS — When to Use Which

**Search in Copilot:** "DeepEval vs RAGAS LLM evaluation framework comparison"

### What to know:

**DeepEval:**
- Lightweight, simpler, easier to start with
- Good for single LLM response evaluation
- Supports many metrics: hallucination, relevance, tone, etc.
- Uses LLM-as-judge (requires API calls)
- Better for: quick iteration, custom metrics, non-RAG use cases

**RAGAS:**
- Built specifically for RAG systems
- Evaluates the whole RAG pipeline: retrieval + generation
- Metrics focus on source-grounded answers
- Can work offline with source documents
- Better for: Phase 3 (RAG project), production RAG systems

### For Phase 2:
Use BOTH, but understand their role:
- **DeepEval** = learn evaluation patterns and metric concepts
- **RAGAS** = learn RAG-specific evaluation (context precision, context recall)

### Overlap and when to use each:

| Scenario | Tool | Why |
|---|---|---|
| Evaluate single Q&A response | DeepEval | Simpler setup |
| Evaluate retrieval quality | RAGAS | Has context_recall, context_precision |
| Evaluate hallucination | Both | DeepEval simpler, RAGAS more thorough |
| Evaluate factuality | RAGAS | More RAG-aware |
| Custom evaluation | DeepEval | Easier to extend |

---

# PART B — DEEPEVAL SETUP AND USAGE

---

## B1. DeepEval Installation and Verification

**Search in Copilot:** "DeepEval installation Python pip setup"

### Step-by-step (at home on your laptop):

```bash
# Create a new folder for Phase 2 work
mkdir phase2_evaluation
cd phase2_evaluation

# Create and activate virtual environment
python -m venv venv
venv\Scripts\activate   # Windows
# source venv/bin/activate  # Mac/Linux

# Install DeepEval
pip install deepeval

# Verify installation
python -c "import deepeval; print(deepeval.__version__)"
```

If you see a version number, installation succeeded.

### ⚠️ Pitfall — Import errors:
If you get `ModuleNotFoundError: No module named deepeval`, you likely:
1. Forgot to activate venv before installing
2. Installed in wrong Python version

Fix: activate venv, reinstall: `pip install deepeval`

### ⚠️ Pitfall — Dependency conflicts:
DeepEval requires newer versions of some libraries (like `pydantic`).
If you get version conflict errors, try:
```bash
pip install --upgrade deepeval
```

---

## B2. DeepEval Hallucination Metric (Core Concept)

**Search in Copilot:** "DeepEval hallucination detection how it works"
**Search in Copilot:** "what is LLM hallucination examples"

### What is hallucination:
A hallucination is a fact/claim in the LLM response that is NOT in the source material and is presented as fact.

**Examples:**
```
Source: "New Delhi is the capital of India."
Response: "New Delhi is the capital of India and was founded in 1911."
Issue: "founded in 1911" is not in source. If true but missing from source = hallucination.

Source: "Python is a programming language created by Guido van Rossum."
Response: "Python was created in 1989 by Guido van Rossum in Netherlands."
Issue: "1989" and "in Netherlands" are not in source document. Hallucinations.
```

### DeepEval hallucination check:

```python
from deepeval.metrics import Hallucination
from deepeval.test_cases import LLMTestCase

# Create a test case
test_case = LLMTestCase(
    input="What is the capital of India?",
    actual_output="New Delhi is the capital of India and was founded in 1911.",
    retrieval_context=["New Delhi is the capital of India."]
)

# Create metric
metric = Hallucination()

# Run evaluation
metric.measure(test_case)
print(f"Hallucination score: {metric.score}")  # 0.0 = no hallucination, 1.0 = severe
print(f"Reason: {metric.reason}")  # Explanation
```

### Understanding the score:
- `score = 0.0` → No hallucinations detected
- `score = 0.5` → Some hallucinations present
- `score = 1.0` → Severe hallucinations

### How it works (conceptually):
1. DeepEval extracts claims from the LLM response
2. Checks if each claim is supported by retrieval_context
3. Scores based on percentage of unsupported claims

### ⚠️ Pitfall — False positives/negatives:
DeepEval uses another LLM (default: GPT-3.5) to detect hallucinations.
LLMs can be wrong.
- False positive: Claims something is a hallucination when it's not
- False negative: Misses actual hallucinations

Example: If source says "AI is powerful" and response says "AI is very powerful", DeepEval might flag "very" as hallucination even though it's reasonable inference.

### ⚠️ Pitfall — Empty retrieval_context:
If `retrieval_context` is empty or not provided, hallucination detection cannot work.
Always provide the source document.

---

## B3. DeepEval Answer Relevance Metric

**Search in Copilot:** "DeepEval answer relevance metric how it measures"

### What it measures:
Does the LLM response actually answer the question asked?

**Examples:**
```
Question: "What is the capital of India?"
Response 1: "New Delhi is the capital of India." → Relevant (answers directly)
Response 2: "India is a large country in South Asia." → Not relevant (doesn't answer)
Response 3: "The capital is New Delhi, which has a population of 20 million." → Relevant (answers + extra info)
```

### DeepEval relevance check:

```python
from deepeval.metrics import AnswerRelevancy
from deepeval.test_cases import LLMTestCase

test_case = LLMTestCase(
    input="What is the capital of India?",
    actual_output="New Delhi is the capital of India."
)

metric = AnswerRelevancy()
metric.measure(test_case)

print(f"Relevance score: {metric.score}")  # 0.0 to 1.0
print(f"Reason: {metric.reason}")
```

### Understanding the score:
- `score >= 0.8` → Highly relevant, directly answers question
- `score 0.5-0.8` → Somewhat relevant, has answer but with noise
- `score < 0.5` → Low relevance, does not answer well

### How it works:
1. DeepEval asks: "Does this response answer the input question?"
2. Uses LLM-as-judge to score
3. Returns score 0.0-1.0

### ⚠️ Pitfall — Ambiguous questions:
If the question is vague, relevance scoring becomes unreliable.

Example:
```
Question: "Tell me about India"  (vague)
Response 1: "Capital is New Delhi" (could be relevant)
Response 2: "Population is 1.4 billion" (could be relevant)
Response 3: "Food is spicy" (could be relevant)
```

DeepEval may score all three differently depending on the LLM judge's interpretation.

### ⚠️ Pitfall — Partial answers:
If the question has multiple parts and response answers only some, relevance score will be lower.

Example:
```
Question: "What is the capital and main port of India?"
Response: "New Delhi is the capital." → Incomplete, scores lower
```

---

## B4. DeepEval with Local Models (Ollama)

**Search in Copilot:** "DeepEval with local models Ollama configuration"

### Why this matters:
DeepEval by default uses OpenAI's GPT models. But you can configure it to use Ollama (local, free).

### Configuration:

```python
from deepeval.models import DeepEvalBaseLLM
from ollama import Client as OllamaClient

class OllamaLLM(DeepEvalBaseLLM):
    def __init__(self, model_name="llama3.1:8b"):
        self.model_name = model_name
        self.client = OllamaClient(host='http://localhost:11434')

    def load_model(self):
        pass  # Ollama already running

    def generate(self, prompt):
        response = self.client.generate(
            model=self.model_name,
            prompt=prompt,
            stream=False
        )
        return response['response']

    async def a_generate(self, prompt):
        return self.generate(prompt)  # Ollama is blocking anyway

# Set as default for DeepEval
from deepeval.models import set_llm
set_llm(OllamaLLM())
```

### Practical consideration:
Using Ollama for DeepEval evaluation means:
- No API costs ✓
- Slower than Groq (local inference) ✗
- Acceptable for learning and development ✓
- NOT recommended for production scale (10,000+ evaluations) ✗

### ⚠️ Pitfall — Ollama must be running:
If Ollama is not running (`ollama serve`), the generate() call hangs.

### ⚠️ Pitfall — Model quality affects evaluation:
Ollama's llama3.1:8b is good but not as good as GPT-4 for judgment tasks.
You may see more false positives/negatives in evaluation scores.
Accept this as a trade-off of using free tools.

---

# PART C — RAGAS FRAMEWORK

---

## C1. RAGAS Installation and Setup

**Search in Copilot:** "RAGAS framework installation Python setup"

### Step-by-step:

```bash
# Make sure venv is activated
pip install ragas

# Verify installation
python -c "import ragas; print(ragas.__version__)"
```

### What is RAGAS:
RAGAS = Retrieval-Augmented Generation Assessment. Built by researchers specifically for evaluating RAG pipelines.

### Why RAGAS for Phase 2:
- You are learning RAG concepts now
- Phase 3 will use RAGAS in your portfolio project
- Understanding RAGAS metrics now = better Phase 3

---

## C2. RAGAS: Faithfulness Metric (Most Important)

**Search in Copilot:** "RAGAS faithfulness metric how it works"
**Search in Copilot:** "faithfulness score LLM evaluation grounded in source"

### What it measures:
Is the LLM response grounded in the provided source documents? Does it stick to the facts in the source?

**Difference from Hallucination:**
- **Hallucination** (DeepEval): Are there claims NOT in source?
- **Faithfulness** (RAGAS): What percentage of the response IS supported by source?

Different perspective, similar outcome.

### Example:
```
Source: "India's capital is New Delhi. It has 20 million people."
Response: "New Delhi is India's capital with 20 million population."

Faithfulness analysis:
- "India's capital is New Delhi" → in source ✓
- "has 20 million people" → in source ✓
- Score: 100% of response supported → faithfulness = 1.0 (perfect)

---

Source: "India's capital is New Delhi."
Response: "New Delhi is India's capital with 20 million people and 500 years old."

Faithfulness analysis:
- "New Delhi is India's capital" → in source ✓
- "20 million people" → not in source ✗
- "500 years old" → not in source ✗
- Score: 33% supported, 67% unsupported → faithfulness = 0.33 (poor)
```

### RAGAS faithfulness code:

```python
from ragas.metrics import faithfulness
from ragas.llm import LangchainLLM
from ragas.embeddings import LangchainEmbeddings

# Setup (detailed in C3)
from langchain.chat_models import ChatOpenAI

llm = ChatOpenAI(model="gpt-3.5-turbo")
ragas_llm = LangchainLLM(llm=llm)

# Your data
question = "What is India's capital?"
answer = "New Delhi is India's capital with 20 million population."
contexts = ["India's capital is New Delhi. It has 20 million people."]

# Calculate faithfulness
score = faithfulness.score(
    question=question,
    answer=answer,
    contexts=contexts,
    llm=ragas_llm
)

print(f"Faithfulness score: {score}")  # 0.0 to 1.0
```

### Understanding the score:
- `score >= 0.8` → Highly faithful, mostly grounded in source
- `score 0.5-0.8` → Mostly faithful with some unsupported claims
- `score < 0.5` → Poor faithfulness, lots of hallucinations

### ⚠️ Pitfall — Requires LLM for scoring:
RAGAS faithfulness uses another LLM call to evaluate. This means:
- Cost: Each evaluation = 2-3 API calls (question + answer + context evaluation)
- Latency: Slower than offline metrics

### ⚠️ Pitfall — Sensitive to source quality:
If your source document is incomplete, RAGAS will penalize answers for information not in source.

Example:
```
Question: "What year was India's capital established?"
Answer: "1911"
Source: "New Delhi is India's capital."  (no year mentioned)

Faithfulness will be low because year not in source,
even though answer is factually correct.
```

Solution: Ensure your source documents are complete.

---

## C3. RAGAS: Answer Relevance Metric

**Search in Copilot:** "RAGAS answer relevance metric score meaning"

### What it measures:
Does the answer address the given question? Similar to DeepEval but RAGAS-optimized for RAG.

### RAGAS answer relevance code:

```python
from ragas.metrics import answer_relevancy

# Same data as before
question = "What is India's capital?"
answer = "New Delhi is India's capital."

score = answer_relevancy.score(
    question=question,
    answer=answer
)

print(f"Answer relevance score: {score}")  # 0.0 to 1.0
```

### Understanding the score:
- `score >= 0.8` → Directly answers the question
- `score 0.5-0.8` → Answers with some irrelevant content
- `score < 0.5` → Does not answer the question

### RAGAS vs DeepEval relevance:
Both measure similar things but:
- **RAGAS** = more contextual, considers RAG pipeline
- **DeepEval** = simpler, one-off relevance check

For Phase 2 learning, they give similar results.

---

## C4. RAGAS: Context Recall Metric

**Search in Copilot:** "RAGAS context recall metric what is it"
**Search in Copilot:** "retrieval evaluation context recall vs precision"

### What it measures:
Out of all the information needed to answer the question, how much was retrieved?

This is **retrieval quality**, not generation quality.

**Example:**
```
Question: "What are the three capitals of India?"
Correct answer: "New Delhi (executive), Kolkata (judicial), Mumbai (financial)"

Retrieved context: "New Delhi is the capital of India."
Missing from retrieval: Kolkata and Mumbai capitals info

Context recall = 1/3 = 0.33 (only 1 out of 3 capitals retrieved)
```

### Why this matters:
In RAG, if retrieval is bad, generation cannot be good.
Context recall tells you: "Did the retriever find the right documents?"

### RAGAS context recall code:

```python
from ragas.metrics import context_recall

question = "What are India's three capitals?"
ground_truth = "New Delhi (executive), Kolkata (judicial), Mumbai (financial)"
contexts = ["New Delhi is the capital of India."]

score = context_recall.score(
    question=question,
    ground_truth=ground_truth,
    contexts=contexts
)

print(f"Context recall score: {score}")  # 0.0 to 1.0
```

### Understanding the score:
- `score = 1.0` → All necessary information retrieved
- `score = 0.5` → Half the necessary information retrieved
- `score = 0.0` → None of necessary information retrieved

### ⚠️ Pitfall — Requires ground truth:
Context recall REQUIRES a reference answer (`ground_truth`).
If you don't have this, you cannot measure context recall.

### ⚠️ Pitfall — Context window limitations:
If the question is complex and needs 10 documents but retriever only returns top 5, context recall will be low.
This is useful signal: "Your retriever needs better ranking."

---

## C5. RAGAS: Context Precision Metric

**Search in Copilot:** "RAGAS context precision metric what does it measure"
**Search in Copilot:** "precision vs recall retrieval evaluation"

### What it measures:
Out of all the retrieved documents, how many were actually relevant to answering the question?

This is the opposite of recall — measure of "cleanliness" of retrieval.

**Example:**
```
Question: "What is the capital of India?"
Retrieved documents:
1. "New Delhi is the capital of India." ✓ Relevant
2. "India is a large country." ✗ Not relevant
3. "New Delhi has a population of 20 million." ✓ Relevant
4. "The Taj Mahal is in Agra." ✗ Not relevant

Context precision = 2/4 = 0.5 (2 relevant out of 4 retrieved)
```

### Why this matters:
High precision = clean retrieval, minimal noise.
If precision is low, your retriever is returning too many irrelevant documents.

### RAGAS context precision code:

```python
from ragas.metrics import context_precision

question = "What is the capital of India?"
ground_truth = "New Delhi"
contexts = [
    "New Delhi is the capital of India.",
    "India is a large country.",
    "New Delhi has a population of 20 million.",
    "The Taj Mahal is in Agra."
]

score = context_precision.score(
    question=question,
    ground_truth=ground_truth,
    contexts=contexts
)

print(f"Context precision score: {score}")  # 0.0 to 1.0
```

### Understanding the score:
- `score = 1.0` → All retrieved documents relevant
- `score = 0.5` → Half are relevant, half are noise
- `score = 0.0` → No retrieved documents relevant

### Recall vs Precision trade-off (IMPORTANT for interviews):

| Scenario | Recall | Precision | Problem |
|---|---|---|---|
| Retriever returns 100 docs | High (found everything) | Low (lots of noise) | Slow, expensive |
| Retriever returns 1 doc | Low (might miss things) | High (if correct) | Might fail to answer |
| Retriever returns 5 smart docs | Medium | High | Optimal |

**Interview answer:** "Context recall and precision are in tension. You need to find the right balance for your use case. For strict factual Q&A, high precision matters. For exploratory search, high recall matters."

---

## C6. RAGAS: Setting Up with OpenAI vs Ollama

**Search in Copilot:** "RAGAS with OpenAI API configuration"
**Search in Copilot:** "RAGAS local LLM Ollama setup"

### Option 1: RAGAS with OpenAI (simpler, better quality):

```python
import os
from langchain.chat_models import ChatOpenAI
from ragas.llm import LangchainLLM
from ragas.embeddings import LangchainEmbeddings

# Set API key (from .env)
os.environ["OPENAI_API_KEY"] = "your_key"

llm = ChatOpenAI(model="gpt-3.5-turbo")
embeddings = LangchainEmbeddings(model="text-embedding-3-small")

ragas_llm = LangchainLLM(llm=llm)
ragas_embeddings = LangchainEmbeddings(embeddings=embeddings)
```

### Option 2: RAGAS with Ollama (free, local):

```python
from langchain.llms import Ollama
from langchain.embeddings import OllamaEmbeddings
from ragas.llm import LangchainLLM

llm = Ollama(model="llama3.1:8b", base_url="http://localhost:11434")
embeddings = OllamaEmbeddings(model="llama3.1:8b")

ragas_llm = LangchainLLM(llm=llm)
ragas_embeddings = OllamaEmbeddings(model_name="llama3.1:8b")
```

### ⚠️ Pitfall — Embeddings are required:
RAGAS metrics use embeddings (vector representations) to compare text similarity.
If you don't set embeddings, metrics fail silently or with cryptic errors.

Always explicitly set embeddings when initializing.

### ⚠️ Pitfall — Ollama is slow for metrics:
If you use Ollama locally, expect 2-5 seconds per metric evaluation.
This is acceptable for Phase 2 learning but not for production.

---

# PART D — UNDERSTANDING TRADE-OFFS (Critical for Interviews)

---

## D1. Cost vs Accuracy Trade-off

**Search in Copilot:** "LLM evaluation framework cost accuracy trade off"

### The problem:
Better evaluation metrics = more accurate scores BUT higher cost and latency.

### Examples:

**Offline metrics (fast, cheap, less accurate):**
```
Rule: If answer contains question's keywords, score = 1.0
Speed: Instant
Cost: Zero
Accuracy: Low (keywords ≠ understanding)
```

**Online metrics with local LLM (medium cost/speed/accuracy):**
```
Use Ollama locally to evaluate
Speed: 2-5 seconds per evaluation
Cost: Zero (except laptop power)
Accuracy: Medium (local models less capable)
```

**Online metrics with GPT-4 (slow, expensive, most accurate):**
```
Use GPT-4-turbo to judge responses
Speed: 3-10 seconds per evaluation
Cost: $0.01-0.05 per evaluation (adds up fast)
Accuracy: High (GPT-4 is strong judge)
```

### For Phase 2:
Use Ollama (free) for learning.
Understand the trade-off: you get lower accuracy, but zero cost.
In Phase 3, you might use Groq (faster) or even GPT-3.5 (better quality).

### Interview answer:
"Evaluation metrics have a cost-accuracy curve. For my project, I used local Ollama models for development because cost mattered more than perfect accuracy. In production, if budget allows, I would use GPT-3.5-turbo for better metric reliability."

---

## D2. Offline vs Online Evaluation Trade-off

**Search in Copilot:** "offline vs online evaluation LLM metrics what is difference"

### Offline evaluation:
- Compares response to source/reference WITHOUT using an LLM
- Examples: keyword matching, embedding similarity, string distance

**Example:**
```python
def offline_faithfulness(answer, source):
    """Check if key terms from answer are in source"""
    answer_terms = set(answer.lower().split())
    source_terms = set(source.lower().split())
    overlap = len(answer_terms & source_terms)
    return overlap / len(answer_terms)

# Fast, no API cost
score = offline_faithfulness("New Delhi is capital", "New Delhi capital")
```

**Pros:** Fast, cheap, deterministic
**Cons:** Shallow, misses semantic understanding

### Online evaluation:
- Uses another LLM call to judge the response
- Examples: DeepEval Hallucination, RAGAS Faithfulness

**Pros:** Deep, semantic understanding, catches subtle issues
**Cons:** Slower, costs money, depends on judge LLM quality

### When to use each:

| Use case | Method | Why |
|---|---|---|
| Quick development iterations | Offline | Fast feedback |
| Large-scale evaluation (10k+ responses) | Offline | Can't afford 10k API calls |
| Production scoring (must be accurate) | Online | Better quality |
| Hybrid (scalable + accurate) | Offline + Online | Offline first pass, online for borderline cases |

### Interview answer:
"Offline metrics are my first pass — fast and cheap. When I need high confidence, I use online metrics with LLM-as-judge. In production, I often use a hybrid: offline for filtering, online for edge cases. This balances cost and accuracy."

---

## D3. Metric Reliability and Failure Modes

**Search in Copilot:** "LLM evaluation metrics unreliability false positives"

### Metric failure modes (common issues):

**1. False positives (metric says bad when answer is good):**
```
Question: "Is Python useful?"
Answer: "Yes, Python is very useful."
Source: "Python is useful for scripting."

Hallucination metric: "very" not in source → Hallucination detected (WRONG)
```

**2. False negatives (metric says good when answer is bad):**
```
Question: "Who is the PM of India?"
Answer: "Rajiv Gandhi is the Prime Minister."
Source: "Rajiv Gandhi was Prime Minister from 1984-1989."

Faithfulness metric: "Rajiv Gandhi" and "Prime Minister" in source → Score 1.0 (WRONG, outdated)
```

**3. Metric manipulation (answer designed to game the metric):**
```
Question: "What is AI?"
Good answer: "AI is intelligence demonstrated by machines."
Gamed answer: "AI intelligence AI demonstrated AI machines AI."

String-based metric counts keyword "AI" → Gamed answer scores higher (WRONG)
```

### Why this happens:
- LLMs used for evaluation can be wrong
- Metrics are heuristics, not ground truth
- Context matters, metrics are simplifications

### Handling this in practice:
```python
# Don't trust single metric
def evaluate_answer(question, answer, source):
    hallucination_score = metric_hallucination(answer, source)
    relevance_score = metric_relevance(question, answer)
    faithfulness_score = metric_faithfulness(answer, source)
    
    # Require agreement across metrics
    if hallucination_score < 0.5 and faithfulness_score < 0.5:
        # Multiple metrics agree → answer is bad
        return False
    elif hallucination_score > 0.8 and faithfulness_score > 0.8:
        # Multiple metrics agree → answer is good
        return True
    else:
        # Metrics disagree → manual review needed
        return "REVIEW_NEEDED"
```

### Interview answer:
"Evaluation metrics are heuristics, not ground truth. I use multiple metrics and require agreement before marking an answer as 'pass'. If metrics disagree, I flag for manual review. This catches metric false positives/negatives."

---

## D4. Hallucination Detection — Edge Cases

**Search in Copilot:** "hallucination detection LLM challenges edge cases"

### Common challenges:

**1. Inference vs Hallucination:**
```
Source: "India is in South Asia. Delhi is a city in India."
Answer: "Delhi is in South Asia."

Is this hallucination? No, it's valid inference from the source.
But metric might flag it if it doesn't find "South Asia" in context of Delhi.
```

**2. Reasonable elaboration:**
```
Source: "New Delhi has a population of 20 million."
Answer: "New Delhi is a major metropolitan area with approximately 20 million residents."

"major metropolitan area" not explicitly in source. Hallucination? Reasonable inference.
```

**3. Typos and variations:**
```
Source: "The capital is New Delhi."
Answer: "New Dilli is the capital."  (typo, should be "Delhi")

Is this a hallucination or just a typo? Metric might treat differently.
```

### Best practice:
Understand that hallucination detection is not binary.
Use scores, not hard pass/fail.
```python
if hallucination_score < 0.3:
    # Likely real hallucination
    action = "REJECT"
elif hallucination_score < 0.7:
    # Borderline, manual review
    action = "REVIEW"
else:
    # Likely safe
    action = "ACCEPT"
```

### Interview answer:
"Hallucination detection is challenging because inference and elaboration can look like hallucinations to automated metrics. I use thresholds instead of binary checks, and I always combine multiple signals before rejecting an answer. Edge cases go to manual review."

---

# PART E — BUILDING YOUR TEST SUITE (Practical Project)

---

## E1. Test Suite Architecture (Planning Phase)

**Search in Copilot:** "Python testing framework test suite structure organization"

### What you will build:
A Python script that:
1. Takes a Q&A dataset (questions + expected answers)
2. Gets LLM responses for each question
3. Evaluates each response using DeepEval and RAGAS metrics
4. Generates a report

### Structure:

```
phase2_eval_project/
├── data/
│   └── sample_qa.json          # Your Q&A dataset
├── evaluator.py                # Main evaluation script
├── results/
│   ├── scores.json             # Metric scores
│   └── report.html             # HTML report
├── requirements.txt
└── .env
```

### Dataset format (JSON):
```json
[
  {
    "question": "What is India's capital?",
    "expected_answer": "New Delhi",
    "source": "New Delhi is the capital of India."
  },
  {
    "question": "What year was Python created?",
    "expected_answer": "1989",
    "source": "Python was created in 1989 by Guido van Rossum."
  }
]
```

---

## E2. Building the Evaluator (Step-by-step)

**Search in Copilot:** "Python script read JSON file process data"

### Step 1: Load data and call LLM

```python
import json
import os
from groq import Groq
from dotenv import load_dotenv

load_dotenv()

# Load your Q&A data
with open("data/sample_qa.json", "r") as f:
    qa_data = json.load(f)

# Initialize Groq client
client = Groq(api_key=os.getenv("GROQ_API_KEY"))

# Get LLM responses
results = []
for item in qa_data:
    response = client.chat.completions.create(
        model="llama-3.1-8b-instant",
        messages=[
            {"role": "user", "content": item["question"]}
        ],
        temperature=0.3,
        max_tokens=200
    )
    
    answer = response.choices[0].message.content
    
    results.append({
        "question": item["question"],
        "expected_answer": item["expected_answer"],
        "source": item["source"],
        "llm_response": answer
    })

print(f"Got {len(results)} LLM responses")
```

### Step 2: Add DeepEval metrics

```python
from deepeval.metrics import Hallucination, AnswerRelevancy
from deepeval.test_cases import LLMTestCase

# Evaluate each response
for result in results:
    # Create test case
    test_case = LLMTestCase(
        input=result["question"],
        actual_output=result["llm_response"],
        retrieval_context=[result["source"]]
    )
    
    # Hallucination metric
    hallucination_metric = Hallucination()
    hallucination_metric.measure(test_case)
    result["hallucination_score"] = hallucination_metric.score
    result["hallucination_reason"] = hallucination_metric.reason
    
    # Relevance metric
    relevance_metric = AnswerRelevancy()
    relevance_metric.measure(test_case)
    result["relevance_score"] = relevance_metric.score
    
    print(f"Question: {result['question'][:50]}")
    print(f"  Hallucination: {result['hallucination_score']:.2f}")
    print(f"  Relevance: {result['relevance_score']:.2f}")
```

### ⚠️ Pitfall — Metric speed:
DeepEval makes LLM calls for each metric on each response.
For 10 questions × 2 metrics = 20 API calls. This takes time.
Be patient. Expect 10-20 seconds total.

### ⚠️ Pitfall — Metric cost:
If using OpenAI, each metric call costs money.
Calculate: 10 questions × 2 metrics × $0.001 per call = $0.02.
Acceptable for learning, but adds up at scale.

---

## E3. Add RAGAS Metrics (If You Have OpenAI Credits)

**Search in Copilot:** "RAGAS integration Python test suite"

### Installation:
```bash
pip install ragas
# Also need: pip install langchain
```

### Code:

```python
from ragas.metrics import faithfulness, answer_relevancy
from langchain.chat_models import ChatOpenAI
from ragas.llm import LangchainLLM

# Setup (uses OpenAI by default)
os.environ["OPENAI_API_KEY"] = os.getenv("OPENAI_API_KEY")
llm = ChatOpenAI(model="gpt-3.5-turbo")
ragas_llm = LangchainLLM(llm=llm)

# Evaluate with RAGAS
for result in results:
    faith_score = faithfulness.score(
        question=result["question"],
        answer=result["llm_response"],
        contexts=[result["source"]],
        llm=ragas_llm
    )
    result["ragas_faithfulness"] = faith_score
    
    rel_score = answer_relevancy.score(
        question=result["question"],
        answer=result["llm_response"],
        llm=ragas_llm
    )
    result["ragas_relevance"] = rel_score

    print(f"RAGAS - Faithfulness: {faith_score:.2f}, Relevance: {rel_score:.2f}")
```

### ⚠️ Pitfall — RAGAS is slow:
RAGAS metrics take 2-5 seconds each because they use LLM calls.
For 10 questions × 2 metrics = 20-50 seconds.

### ⚠️ Pitfall — Dependency issues:
RAGAS has specific dependency versions. If you get import errors:
```bash
pip install --upgrade ragas langchain
```

---

## E4. Generate Report and Save Results

```python
import json
from datetime import datetime

# Save detailed results as JSON
output = {
    "timestamp": datetime.now().isoformat(),
    "total_questions": len(results),
    "metrics": {
        "avg_hallucination_score": sum(r["hallucination_score"] for r in results) / len(results),
        "avg_relevance_score": sum(r["relevance_score"] for r in results) / len(results),
    },
    "details": results
}

with open("results/scores.json", "w") as f:
    json.dump(output, f, indent=2)

# Simple HTML report
html = f"""
<html>
<head><title>Evaluation Report</title></head>
<body>
<h1>LLM Evaluation Report</h1>
<p>Generated: {output['timestamp']}</p>
<p>Total questions: {output['total_questions']}</p>
<h2>Average Scores</h2>
<ul>
<li>Hallucination: {output['metrics']['avg_hallucination_score']:.2f}</li>
<li>Relevance: {output['metrics']['avg_relevance_score']:.2f}</li>
</ul>
<h2>Details</h2>
<table border="1">
<tr><th>Question</th><th>Hallucination</th><th>Relevance</th></tr>
"""

for r in results:
    html += f"<tr><td>{r['question'][:50]}</td><td>{r['hallucination_score']:.2f}</td><td>{r['relevance_score']:.2f}</td></tr>"

html += """
</table>
</body>
</html>
"""

with open("results/report.html", "w") as f:
    f.write(html)

print("Results saved to results/scores.json and results/report.html")
```

---

# PART F — PHASE 2 EXIT PROJECT (Your Deliverable)

---

## F1. Complete Evaluator Script (Full Code)

Here is the complete script you should build and run:

```python
"""
Phase 2 Exit Project — LLM Evaluation Test Suite
Tests Q&A responses using DeepEval and RAGAS metrics
"""

import json
import os
import time
from datetime import datetime
from groq import Groq
from dotenv import load_dotenv

# Try to import evaluation tools (handle missing gracefully)
try:
    from deepeval.metrics import Hallucination, AnswerRelevancy
    from deepeval.test_cases import LLMTestCase
    DEEPEVAL_AVAILABLE = True
except ImportError:
    DEEPEVAL_AVAILABLE = False
    print("⚠️  DeepEval not installed. Install with: pip install deepeval")

try:
    from ragas.metrics import faithfulness, answer_relevancy as ragas_answer_relevancy
    from langchain.chat_models import ChatOpenAI
    from ragas.llm import LangchainLLM
    RAGAS_AVAILABLE = False  # Skip for now, requires OpenAI key
except ImportError:
    RAGAS_AVAILABLE = False
    print("Note: RAGAS not installed. Optional for Phase 2.")

load_dotenv()

class LLMEvaluator:
    def __init__(self):
        self.client = Groq(api_key=os.getenv("GROQ_API_KEY"))
        self.results = []
    
    def load_qa_data(self, file_path):
        """Load Q&A dataset from JSON"""
        with open(file_path, "r") as f:
            return json.load(f)
    
    def get_llm_response(self, question):
        """Call LLM to answer question"""
        response = self.client.chat.completions.create(
            model="llama-3.1-8b-instant",
            messages=[
                {
                    "role": "system",
                    "content": "You are a helpful assistant. Answer concisely."
                },
                {
                    "role": "user",
                    "content": question
                }
            ],
            temperature=0.3,
            max_tokens=200
        )
        return response.choices[0].message.content
    
    def evaluate_with_deepeval(self, question, answer, source):
        """Evaluate using DeepEval metrics"""
        if not DEEPEVAL_AVAILABLE:
            return {}
        
        test_case = LLMTestCase(
            input=question,
            actual_output=answer,
            retrieval_context=[source]
        )
        
        results = {}
        
        try:
            # Hallucination check
            metric = Hallucination()
            metric.measure(test_case)
            results["hallucination_score"] = metric.score
            results["hallucination_reason"] = metric.reason
        except Exception as e:
            results["hallucination_error"] = str(e)
        
        try:
            # Relevance check
            metric = AnswerRelevancy()
            metric.measure(test_case)
            results["relevance_score"] = metric.score
        except Exception as e:
            results["relevance_error"] = str(e)
        
        return results
    
    def evaluate_batch(self, qa_data):
        """Evaluate all Q&A pairs"""
        print(f"\nEvaluating {len(qa_data)} questions...")
        print("-" * 60)
        
        for i, item in enumerate(qa_data, 1):
            question = item["question"]
            source = item.get("source", "")
            
            print(f"\n[{i}/{len(qa_data)}] {question[:60]}...")
            
            # Get LLM response
            llm_response = self.get_llm_response(question)
            print(f"LLM: {llm_response[:80]}...")
            
            # Evaluate
            eval_results = self.evaluate_with_deepeval(question, llm_response, source)
            
            result = {
                "question": question,
                "source": source,
                "llm_response": llm_response,
                **eval_results
            }
            
            self.results.append(result)
            
            # Print scores
            if "hallucination_score" in eval_results:
                print(f"  Hallucination: {eval_results['hallucination_score']:.2f}")
            if "relevance_score" in eval_results:
                print(f"  Relevance: {eval_results['relevance_score']:.2f}")
            
            time.sleep(1)  # Be nice to API
    
    def generate_report(self, output_dir="results"):
        """Generate JSON and HTML reports"""
        os.makedirs(output_dir, exist_ok=True)
        
        # Calculate summaries
        hallucination_scores = [r["hallucination_score"] for r in self.results 
                               if "hallucination_score" in r]
        relevance_scores = [r["relevance_score"] for r in self.results 
                           if "relevance_score" in r]
        
        summary = {
            "timestamp": datetime.now().isoformat(),
            "total_evaluated": len(self.results),
            "avg_hallucination": sum(hallucination_scores) / len(hallucination_scores) if hallucination_scores else None,
            "avg_relevance": sum(relevance_scores) / len(relevance_scores) if relevance_scores else None,
        }
        
        # Save JSON
        json_path = os.path.join(output_dir, "scores.json")
        with open(json_path, "w") as f:
            json.dump({
                "summary": summary,
                "details": self.results
            }, f, indent=2)
        print(f"\n✓ Results saved to {json_path}")
        
        # Save HTML report
        html_path = os.path.join(output_dir, "report.html")
        html = self._generate_html(summary)
        with open(html_path, "w") as f:
            f.write(html)
        print(f"✓ Report saved to {html_path}")
    
    def _generate_html(self, summary):
        """Generate HTML report"""
        html = f"""
        <html>
        <head>
            <title>LLM Evaluation Report</title>
            <style>
                body {{ font-family: Arial; margin: 20px; }}
                table {{ border-collapse: collapse; width: 100%; }}
                th, td {{ border: 1px solid #ddd; padding: 8px; text-align: left; }}
                th {{ background-color: #4CAF50; color: white; }}
                .summary {{ background-color: #f0f0f0; padding: 15px; border-radius: 5px; }}
            </style>
        </head>
        <body>
            <h1>LLM Evaluation Report</h1>
            <div class="summary">
                <p><strong>Generated:</strong> {summary['timestamp']}</p>
                <p><strong>Total Evaluated:</strong> {summary['total_evaluated']}</p>
                <p><strong>Avg Hallucination Score:</strong> {summary['avg_hallucination']:.2f if summary['avg_hallucination'] else 'N/A'}</p>
                <p><strong>Avg Relevance Score:</strong> {summary['avg_relevance']:.2f if summary['avg_relevance'] else 'N/A'}</p>
            </div>
            <h2>Detailed Results</h2>
            <table>
                <tr>
                    <th>Question</th>
                    <th>LLM Response</th>
                    <th>Hallucination</th>
                    <th>Relevance</th>
                </tr>
        """
        
        for r in self.results:
            hall = f"{r.get('hallucination_score', 'N/A'):.2f}" if "hallucination_score" in r else "N/A"
            rel = f"{r.get('relevance_score', 'N/A'):.2f}" if "relevance_score" in r else "N/A"
            
            html += f"""
                <tr>
                    <td>{r['question'][:50]}</td>
                    <td>{r['llm_response'][:80]}</td>
                    <td>{hall}</td>
                    <td>{rel}</td>
                </tr>
            """
        
        html += """
            </table>
        </body>
        </html>
        """
        return html


def main():
    # Create sample Q&A data if needed
    sample_data = [
        {
            "question": "What is the capital of India?",
            "source": "New Delhi is the capital of India."
        },
        {
            "question": "What programming language is Python?",
            "source": "Python is a high-level, interpreted programming language."
        },
        {
            "question": "When was Python created?",
            "source": "Python was created in 1989 by Guido van Rossum."
        }
    ]
    
    os.makedirs("data", exist_ok=True)
    with open("data/sample_qa.json", "w") as f:
        json.dump(sample_data, f, indent=2)
    
    # Run evaluation
    evaluator = LLMEvaluator()
    qa_data = evaluator.load_qa_data("data/sample_qa.json")
    evaluator.evaluate_batch(qa_data)
    evaluator.generate_report()
    
    print("\n" + "="*60)
    print("PHASE 2 EXIT PROJECT COMPLETE")
    print("="*60)


if __name__ == "__main__":
    main()
```

---

## F2. Running the Full Script

At home, in your venv:

```bash
# Create data directory
mkdir -p data results

# Run script
python evaluator.py
```

Output:
```
[1/3] What is the capital of India?...
LLM: New Delhi is the capital and the center of India's government...
  Hallucination: 0.15
  Relevance: 0.92

[2/3] What programming language is Python?...
...

✓ Results saved to results/scores.json
✓ Report saved to results/report.html
```

---

# PHASE 2 — TOPIC CHECKLIST

### Concepts (Part A)
- [ ] A1. What is LLM evaluation (conceptually)
- [ ] A2. DeepEval vs RAGAS — when to use which

### DeepEval (Part B)
- [ ] B1. Installation and verification
- [ ] B2. Hallucination metric — concept and code
- [ ] B3. Answer Relevance metric
- [ ] B4. Using Ollama with DeepEval (optional)

### RAGAS (Part C)
- [ ] C1. Installation
- [ ] C2. Faithfulness metric — concept and code
- [ ] C3. Answer Relevance metric (RAGAS version)
- [ ] C4. Context Recall metric
- [ ] C5. Context Precision metric
- [ ] C6. RAGAS setup with OpenAI vs Ollama

### Trade-offs (Part D — Critical for Interviews)
- [ ] D1. Cost vs Accuracy trade-off
- [ ] D2. Offline vs Online evaluation
- [ ] D3. Metric reliability and failure modes
- [ ] D4. Hallucination detection — edge cases

### Building Your Suite (Part E)
- [ ] E1. Test suite architecture and planning
- [ ] E2. Load LLM responses
- [ ] E3. Add DeepEval metrics
- [ ] E4. Add RAGAS metrics (optional)
- [ ] E5. Generate reports

### Exit Project (Part F)
- [ ] F1. Run complete evaluator script
- [ ] F2. Generate JSON and HTML reports

---

# PHASE 2 EXIT CONDITION (Self-test)

You are ready for Phase 3 when you can answer these:

1. What is the difference between hallucination and faithfulness metrics?
2. Explain context recall vs context precision in your own words.
3. Why might a high hallucination score be a false positive?
4. When would you use offline evaluation vs online evaluation?
5. Show your evaluator script and explain how metrics are calculated.
6. What are the cost implications of using GPT-4 vs Ollama for evaluation?
7. Design an evaluation strategy for a BFSI chatbot (financial domain).
8. How would you handle metric disagreement (one metric says good, one says bad)?

---

# COMMON PHASE 2 PITFALLS (Reference)

| Pitfall | Why it happens | How to avoid |
|---|---|---|
| Metrics contradict each other | Each metric measures different aspects | Use multiple metrics, require agreement |
| Evaluation is slow | LLM calls are slow | Use offline metrics first, online for validation |
| Metric scores don't match intuition | Metrics are heuristics, can fail | Add manual review for borderline cases |
| API key accidentally exposed | Copy-paste from .env | Use dotenv, add .env to .gitignore |
| DeepEval fails silently | Wrong LLM configuration | Always set LLM and embeddings explicitly |
| Ollama evaluation hangs | Ollama not running | Start `ollama serve` before running |
| RAGAS requires ground truth | Context recall needs reference answer | Use offline metrics if no ground truth |
| Memory error running metrics | Too many questions at once | Process in batches of 5-10 |

---

# CONNECTING TO YOUR INTERVIEW STORY

By end of Phase 2, you can say:

> "I built an automated test suite for evaluating LLM responses using DeepEval and RAGAS metrics. I understood the trade-offs: hallucination detection is probabilistic, context precision vs recall requires tuning, and cost-accuracy is a curve. I designed a hybrid approach — offline metrics for speed, online metrics for accuracy, and manual review for edge cases. This is how you test non-deterministic systems."

That answer passes Phase 2. That answer opens Phase 3.

---

*Last updated: April 2026 | Phase 2 of 5 | AI-Augmented SDET Roadmap*
