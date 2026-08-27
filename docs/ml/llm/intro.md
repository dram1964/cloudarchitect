# Large Language Models

LLMs are trained on large text sources and learn everything they
read and also learn the structure and grammar of language. As a 
result they can perform language-based tasks, such as answering 
questions, translating from one language to another.

There are three main types of models: 

1. Base: takes a sequence of inputs and predicts what will come next (e.g. predictive text). 
2. Chat (Instruct): works with prompts, to provide outputs. Derived from base models by training it with questions and answers, so that eventually it would predict an answer. 
3. Reasoning: derived from Chat models by instructing the model to 'think step-by-step' and training it with reasoning steps. 

A fourth type is 'Hybrid' models, which vary the balance between 'reasoning' 
and 'chat' mode, depending on the question. The amount of reasoning can also
be changed by 'budget forcing'. Budget forcing attempts to increase the 
amount of reasoning the model uses. This can be done by inserting the word 
'wait' into a reasoning sequence. Because the model is trained to predict 
the next sequence of words, 'wait' makes the model think something like: 
'Wait, have I thought this through properly' and thus generate additional
reasoning steps. 

Chat models still work by predicting the next word: then the new output is
fed back to the model to predicting the next word, and so on until it produces
the final answer. Emergent intelligence is really just a by-product of 
next token prediction: the models are so good at predicting the next token
that they can solve problems at PhD level. 

Chat models, have less 'reasoning' overhead and therefore tend to respond 
faster and are better for interactive use cases. They also 
appear to be better at producing creative content: perhaps because they
apply less reasoning. 

Base models are best for re-training with a new skill. 

LLMs are large - their size is measured by the number of parameters 
it has. For LLMs, parameters act like 
instructions and rules that tell the model how to perform a task. 
The more parameters a model has, the more data it can be fed during training:
this is ofter referred to as training-time scaling. Inference-time scaling
refers to techniques used during inference to improve the performance of 
the model, such as response-augmented generation (RAG). 

LLMs are general purpose - they are pre-trained on a wide variety of text data 
rather than a specific topic. We can then fine-tune the model to perform 
specific tasks by using domain-specific datasets. 

- Few-shot: minimal data is used to fine-tune a model
- Zero-shot: no data used to fine-tune a model

## Frontier Models

1. GPT - OpenAI
2. Claude - Anthropic. Comes in three sizes from smaller to larger: Haiku, Sonnet and Opus.
3. Gemini - Google (closed-source version of Gemma)
4. Grok - x.ai

## Open Source Models

1. Llama - Meta
2. Mixtral - Mistral
3. Qwen - Alibaba Cloud
4. Gemma - Google
5. Phi - Microsoft
6. Deepseek - Deepseek AI
7. GPT-OSS - OpenAI

## Neural Networks

At the heart of LLMs are neural networks which are used 
to mimic the functioning of the human brain. 

The network structure
consists of nodes (neurons) that each implement a mathematical operation 
or model on its input to produce an output. In effect, the neural network
is a network of smaller models working together to produce an output. The
outputs from each neuron can be distorted by a 'non-linearity' or 'activation'
function to ensure that they are each making a unique contribution to the 
overall result. These results are then blended through several layers, to 
produce the final output. 

Transformers are type of neural network, originally developed
by Google Research: see research paper 'Attention is all you need' published
in 2017.  The strength of the connections (weights) between
neurons is adjusted during training, to provide the most accurate output by
reducing the difference between the predicted output and the actual output. 
The attention mechanism enables a model to weigh the 
importance of different tokens in an input sequence when producing output. 
As each token is processed, the weights assigned to surrounding tokens 
are considered when producing the output. 

Adding an Attention Layer to the neural networks used by LLMs, 
became known as the Transformer Architecture. 
The Attention Layer is an optimisation that 
allowed to the neural networks to become ever-larger whilst maintaining
performance. 

Deep Learning refers to the size of the neural networks used by the 
resulting LLMs. 

Self-attention is a mechanism that computes relationships within a single input 
sequence and captures dependancies and contextual information. 

The Transformer Architecture uses an Encoder/Decoder layout to 
process all the tokens simultaneously. 

### Encoder Block

Responsible for creating a contextualised representation of the input. 

The first step is to create the input embeddings (numerical representation of the 
input tokens): 

1. Break down the input into tokens (using a tokenisation strategy). 
See https://platform.openai.com/tokeniser for a working example. Typically, 
for English language inputs, a token is about 4-characters or 0.75 words. So 750 tokens would become 1000 tokens. Numbers are broken-down into 3-digit sections, with a corresponding token for each 3-digit combination.
    - See [Tokenising](#tokenising) for an example in Python.  
2. Create input embeddings 
    - using a pre-defined vocabulary to map tokens to a numeric representation 
    - The mapping between tokens and the numeric representation is referred to as the encoders vocabulary. 
3. Retrieve the Embedding Model
    - maps the tokens to a vector representation
    - vectors encode semantic and syntactic information for each token 
    - similar words are closer to each other in the vector space
    
Positional Encoding is used to add a number to indicate the position of 
each token in the input. Padding or Truncation is used to ensure that all 
input sequences have the same length (required by certain models). 


Input embeddings and Positional Encoding vectors are then fed to the 
Encoder Block of the transformer. The Encoder Block contains:

1. Multi-head Attention Layer (Self-Attention Mechanism)
    - weighs the importance of different tokens
    - provides attention vectors to capture contextual relationships between tokens
    - creates three vectors
        - Query Vector
            - represents each tokens relationship (question) to the other tokens in the sequence
        - Key Vector
            - hold information about all the other tokens in the sequence
        - Value Vector
            - hold information about the current token
    - Similarities between query and key vectors are calculated as 'dot' products to produce 
    similarity scores for each token.  For numerical stability, 
    similarity scores are scaled by dividing by the square-root of the dimensions 
    of the key vectors. 
    - Attention Scores are produced using the Softmax function to normalise the scaled similarity scores
    - The Value Vector is then multiplied by the Attention Score to produce an Attention Vector for each 
    token
    - Attention Vectors are calculated separately on multiple attention heads, focussing on different characteristics. 
    - The Attentions heads are concatenated and transformed to produce the final output of this layer
    
2. Feed Forward Layer
    - models complex relationships within the input sequence
    - processes and transforms information from the Multi-Head Attention mechanism
    - generates context-aware representations for each token
        - linear transformation (learned-weight matrix applied to each representation)
        - activation function
        - further linear transformation reducing dimensionality
    - transformations are applied to individually to each token, so can be executed in parallel
    - results in final representation for each input token that is more compact than original input

### Decoder Block

Responsible for iteratively decoding the encoders output together with the decoders output so far. 

Embeddings are created from the desired output that we want the model to learn using the same process for creating input embeddings. Output Embeddings and Input Embeddings are then fed to the Decoder Block:

1. Masked Multi-Head Attention
    - sees the attention vectors for the input embeddings (cross-attention)
    - only sees the embeddings for the words that come before the 
    currently attended word in the output embeddings (self-attention)
    - the masked multi-head attention must learn what the next word should be
    
2. Multi-Head Attention Layer
    - receives key and value vectors from encoder
    - receives encodings from the masked multi-head decoder layer (output tokens)
    - calculates attention scores between the current output token and the encoder
    - results in a context vector representing the relevant vectors from the input which should be used when calculating the ouput token
    - the context vector is passed to the decoder feed-forward layer
    - linear and softmax layers are then used to format the output and create output probabilities

- Convolutional Neural Networks 
    - vision classification
- Recurrent Neural Networks 
    - pre-dates transformer architectures 
    - Uses the preceding text token as context for the current token
    - Struggles when input text is long, as previous context vanishes
    - can't be parallelised, as context is lost
- Transformers
    - can pay attention to the most important words in the text to retain context
    - Transformer architecture is key to the operation of LLMs. 

## The Illusion of Memory

When using an API to interact with a LLM, each call is stateless: it has no
relation to previous calls that were made. However, you can inform the LLM of
previous responses using an `assistant` role in your message, to represent 
a previous response from the LLM: 

```python
messages = [
    {'role': 'system', 'content': 'You are a helpful assistant'},
    {'role': 'user', 'content': 'Hi, my name is David'},
    {'role': 'assistant', 'content': 'Hi, David. How can I help you today?'},
    {'role': 'user', 'content': 'What is my name?'},
]
```

This is how the on-line models create the 'illusion of memory' - each question 
you send the model in a conversation is pre-pended to any subsequent questions. 
As the conversation develops, each question in a conversation is charged for 
tokens in the current question, plus the tokens from the previous questions:
the 'illusion of memory' comes with a cost. 

## Tokenising

Tokenising involves mapping chunks of input data to numeric codes representing
each chunk. Different models have different 'vocabularies' or encodings for 
these chunks. You can run the tokenising process in Python using the 
`tiktoken` module: 

```python
import tiktoken

encoding = tiktoken.encoding_for_model("gpt-4.1-mini")
tokens = encoding.encode("Henry VIII had six wives")

tokens
```
You can also use `tiktoken` to decode the tokens: 

```python
for token_id in tokens:
    token_text = encoding.decode([token_id])
    print(f"[{token_id}] = [{token_text}]")
```

Notice how whole words after the first word, have a leading space character. 

