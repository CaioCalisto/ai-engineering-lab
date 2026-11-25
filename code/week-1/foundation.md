Read Article ["How LLM Inference Actually Works: From Prompt Tokens to the Next Token"](https://huggingface.co/blog/prismberry-technologies/how-llm-inference-actually-work)

At a high level:

Text Prompt &rarr; Tokenizer &rarr; Token IDs &rarr; Embeddings + Position Information &rarr; Transformer Layers &rarr; Final Hidden State &rarr; Language Modeling Head &rarr; Logits &rarr; Sampling / Decoding &rarr; Next Token &rarr; Append Token to Context &rarr; Repeat

## Training
Modifies the model, weights are updated

## Inference
Uses the model, "What token should come next"

## 1. Tokenizer
- Hugging Face Transformers provides AutoTokenizer, which automatically loads the tokenizer associated with a model.
- Tokenize text and output looks like [504, 1234, 315, 1201, 13, 4567, 1256]. These integers are Token IDs.
- The exact IDs depend on the tokenizer
- 1 token IS NOT 1 word. A token can be a word, part of a word, or even punctuation. For example, "Hello" might be one token, while "Hello!" could be two tokens: "Hello" and "!".

## 2. Chat Template
- Chat models were trained to recognize particular conversational formats.
- Giving an instruction-tuned model incorrectly formatted input can significantly affect its output.
- Pipeline is closer to: Conversation &rarr; Chat Template &rarr; Formatted Text &rarr; Tokenizer &rarr; Token IDs

## 3. Token IDs become Embeddings Vectors
- A token ID is an index into an embedding matrix, which is a large table of numbers that represents the meaning of each token in a high-dimensional space.
- Token ID: 1256 &rarr; [0.17, -0.42, 0.08, ..., 0.31]

## 4. Position Information
- A Transformer therefore needs information about position.
- Position embeddings are added to the token embeddings to give the model information about the order of tokens in the sequence.

## 5. Transformer Layers
- The Transformer architecture consists of multiple layers, each containing self-attention mechanisms and feed-forward neural
- A modern LLM may contain tens or hundreds of Transformer layers.

## 5.1 Self-Attention
- Self-attention allows the model to weigh the importance of different tokens in the input sequence when
- For every token, the model produces three projections: Query, Key, and Value vectors. The attention mechanism computes a weighted sum of the Value vectors based on the similarity between the Query and Key vectors.
- Query asks: What information am I looking for?
- Keys indicate: What information does each token contain?
- Values contain: What information should actually be passed forward?
- The dot product between queries and keys produces attention scores.
- After scaling and normalization, these scores determine how strongly each position should incorporate information from other positions.

## 6. Why casual Language Models need a Mask?
- When predicting token t, a causal language model must not look into future tokens.
- The attention mask is a binary matrix that indicates which tokens can attend to which other tokens. For causal models, the mask ensures that each token can only attend to itself and previous tokens, preventing information leakage from future tokens.

## 7. Multi-Head Attention
- Attention is split across multiple heads.
- Multi-head attention allows the model to focus on different parts of the input sequence simultaneously.
- Each head learns different attention patterns, enabling the model to capture various relationships and dependencies between tokens

## 8. Feed-Forward Neural Networks
- Commonly called the MLP or FFN, this component consists of two linear transformations with a non-linear activation function in between.
- The feed-forward network processes each token's representation independently, allowing the model to learn complex transformations
- Instead of completely replacing the incoming representation, the block adds transformed information back into it:
- - [ x_{new} = x + Attention(x) ]
- - and later:
- - [ x_{newer} = x_{new} + MLP(x_{new}) ]

## 9. The Prefill Phase
- The prefill phase is the initial step in the inference process where the model processes the input tokens to generate a context representation.
- During this phase, the model computes the hidden states for all input tokens, which will be used to predict the next token in the sequence.
- The prefill phase is crucial for establishing the context that the model will use to generate coherent and contextually relevant outputs. It involves passing the token embeddings through the transformer layers, applying self-attention, and updating the hidden states accordingly. This phase sets the stage for the subsequent token generation steps, ensuring that the model has a comprehensive understanding of the input sequence before making predictions.

## 10. LM Head
- The Language Modeling (LM) head is the final component of the model that takes the hidden
- The LM head is responsible for converting the final hidden states produced by the transformer layers into logits, which represent the model's predictions for the next token in the sequence. 
- It typically consists of a linear layer that maps the high-dimensional hidden states to a vocabulary-sized output space, followed by a softmax function to produce probabilities for each token in the vocabulary. The LM head allows the model to generate coherent and contextually appropriate text by selecting the most likely next token based on the learned representations and context established during the prefill phase.

## 11. Seeing the Next-Token Logits With Hugging Face
- The Hugging Face Transformers library provides a convenient way to access the next-token logits during inference.
- Logits are the raw, unnormalized scores output by the model before applying the softmax function to convert them into probabilities. These logits indicate the model's confidence in predicting each token in the vocabulary as the next token in the sequence.

## 12. Softmax
- Softmax is applied to the logits to obtain a probability distribution over the vocabulary, allowing for sampling or selecting the most likely next token. By examining the logits, users can gain insights into the model's decision-making process and understand which tokens are considered more probable given the current context.

## 13. Greedy Decoding
- Greedy decoding is a simple decoding strategy where the model selects the token with the highest probability

## 14. Temperature
- Temperature is a hyperparameter that controls the randomness of the model's predictions during sampling. A higher temperature value (e.g., >1) makes the model's output more diverse and exploratory, while a lower temperature value (e.g., <1) makes the output more deterministic and focused on high-probability tokens. Adjusting the temperature allows users to balance between creativity and coherence in the generated text.
- Higher temperature = more creative, lower temperature = more deterministic
- Lower temperature = more deterministic and focused on high-probability tokens, while a higher temperature allows for more exploration and diversity in the generated text.

## 15. Top-K and Top-P Sampling
- Top-k sampling is a decoding strategy that restricts the model's token selection to the top
- Top-K keeps only the k most probable tokens and redistributes their probabilities, effectively ignoring less likely tokens. This approach helps to maintain a balance between randomness and coherence in the generated text, allowing for more controlled and contextually relevant outputs.
- Top-P works differently, where it considers the cumulative probability distribution of the tokens and selects the smallest set of tokens whose cumulative probability exceeds a specified threshold p. This method allows for dynamic token selection based on the model's confidence, enabling more flexible and contextually appropriate text generation. Both top-k and top-p sampling techniques are commonly used to enhance the quality and diversity of generated text while maintaining coherence with the input context.

## 16. KV Cache
- The KV cache (Key-Value cache) is a mechanism used in transformer models to store
- Instead of recalculating keys and values for all previous tokens during each inference step, the KV cache allows the model to reuse previously computed key and value representations. This significantly speeds up the inference process, especially for long sequences, by reducing redundant computations and enabling efficient token generation. The KV cache is particularly useful in scenarios where the model needs to generate multiple tokens in a sequence, as it maintains the context established by earlier tokens while allowing for rapid generation of subsequent tokens.

### Comparison:
- Naive Generation: Reprocess repeatidaly the entire context for each new token, which is inefficient.
- KV Cache: Store the keys and values for previous tokens to avoid recomputation, improving efficiency

## 17. Decoding Text back to Humans
- The tokenizer reverses the tokenization process, converting token IDs back into human-readable text. This step is essential for presenting the model's output in a format that users can understand and interact with. The decoding process ensures that the generated tokens are transformed into coherent sentences or phrases, maintaining the context and meaning established during the inference process.

## 18. Why streaming feels faster
- When using ChatGPT-like interfaces, text often appears token by token.
- The system does not necessarily wait for the entire response to finish.
- How long does it take before the first generated token becomes available? This is strongly affected by the prefill phase.

## 19. A complete Mental Model
1. Text Prompt: "Explain why the sky appears"
2. Input Processing: The chat system formats the conversation, Tokenizer converts the resulting text into Token IDs.
3. Embeddings: Token IDs index into learned embeddings vectors, position information is incorporated.
4. Transformer Processing: Representations move through multiple decoder layers. Inside those layers, self-attention allows each position to incorporate relevant context while causal constraints prevent access to future positions.
5. Vocabulary Projection: The final relevant hidden state passes through the language-modeling head.
6. Decode: Logits may be modified by temperature, penalties or filtering mechanisms. The next token is sampled from the resulting distribution.
7. Cache update: New attention keys and values are stored, on the next decode step, earlier K/V state are reused rather than fully recalculated.
8. Repeat: The new token is appended to the context, and the process repeats until a stopping criterion is met (e.g., end-of-sequence token, maximum length, or user-defined conditions).

