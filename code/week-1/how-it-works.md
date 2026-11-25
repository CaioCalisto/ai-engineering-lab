
# How Text Is Processed in a Large Language Model (LLM)

## 1. Tokenization

A Large Language Model (LLM) does not process text directly as humans read it. First, the input text is divided into smaller units called **tokens**.

A token can represent:
- A complete word.
- Part of a word.
- A punctuation mark.
- A space or other text element.

**Example**

Input text:

> Hello world

Possible tokenization:
- `Hello`
- ` world`

Note: The exact tokenization depends on the tokenizer used by the model.

---

## 2. Token IDs

Each token is assigned a numerical identifier called a **Token ID**.

These IDs are integers used by the model to identify tokens within its vocabulary.

**Example**

| Token | Token ID |
|---|---:|
| Hello | 892 |
| world | 453 |
| AI | 1532 |
| is | 78 |

**Important:** Token IDs are identifiers, not numerical representations of the meaning of words.

---

## 3. Embedding Matrix

The model uses the Token ID to retrieve a corresponding vector from a structure called the **Embedding Matrix**.

The Embedding Matrix is a numerical matrix in which:

- Each row corresponds to a token in the vocabulary.
- Each column represents one dimension of the embedding vector.
- Each row contains the numerical representation associated with that token.

**Example**

Imagine a model with:

- Vocabulary size: 50,000 tokens.
- Embedding dimension: 4,096.

The Embedding Matrix would have the following dimensions:

`50,000 × 4,096`

This means:

- 50,000 rows (one per token).
- 4,096 columns (dimensions per embedding vector).

---

## 4. Embeddings

When the model looks up a Token ID in the Embedding Matrix, it retrieves a numerical vector called an **Embedding**.

**Example**

- Token: `Hello`
- Token ID: `892`
- Embedding (illustrative):

`[0.21, -0.45, 0.82, 0.13, ...]`

These vectors are learned during model training and encode useful information about tokens, including semantic and linguistic relationships.

The Transformer processes these numerical representations rather than the original text.

---

## 5. Positional Information

Transformers also need information about the position of each token in the input sequence.

For example:

- "The dog chased the cat."
- "The cat chased the dog."

The same words appear, but their order changes the meaning.

Positional information allows the Transformer to take token positions into account.

Depending on the architecture, this information can be incorporated through positional embeddings, positional encodings, or mechanisms such as RoPE (Rotary Position Embeddings).

---

## 6. Complete Processing Pipeline

The process can be summarized as follows:

```text
INPUT TEXT
    |
    v
TOKENIZER
    |
    | Splits text into tokens
    v
TOKENS
    |
    v
TOKEN IDs
    |
    | Numerical identifiers
    v
EMBEDDING MATRIX
    |
    | Retrieves corresponding vectors
    v
EMBEDDINGS
    |
    v
POSITIONAL INFORMATION
    |
    | Incorporates token position
    v
TRANSFORMER
    |
    | Attention + Feed-Forward Networks
    v
MODEL PROCESSING
```

---

## 7. Key Concepts to Remember

| Concept | Definition |
|---|---|
| Token | A unit of text processed by the model. |
| Tokenizer | The component responsible for splitting text into tokens. |
| Token ID | A numerical identifier assigned to each token. |
| Embedding Matrix | A matrix containing the learned numerical vectors associated with the vocabulary. |
| Embedding | The numerical vector representation retrieved for a token. |
| Positional Information | Information that allows the model to account for token order and position. |
| Transformer | The neural network architecture that processes these representations. |

---

## 8. Final Summary

An LLM converts input text into tokens, assigns each token a numerical ID, uses those IDs to retrieve vectors from the Embedding Matrix, incorporates positional information, and passes the resulting representations to the Transformer architecture for further processing.

The key distinction:

- **Tokens** are units of text.
- **Token IDs** identify those tokens numerically.
- **Embeddings** provide their learned numerical representations.
- **Positional Information** helps the model understand the order of tokens.
- **The Transformer** processes these representations to understand context and generate output.
