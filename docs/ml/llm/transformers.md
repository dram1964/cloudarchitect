# Transformers

Transformers are type of neural network, originally developed
by Google Research. 
    
## Attention

see research paper 'Attention is all you need'

The attention mechanisms enables a model to weigh the importance of different
tokens in an input sequence when producing output. As each token is processed, 
the weights assigned to surrounding tokens are considered when producing the output. 

Self attention is a mechanism that computes relationships within a single input 
sequence and captures dependancies and contextual information. 

## Transformer Architecture

Uses an Encoder/Decoder architecture to process all the tokens simultaneously. 

### Encoder Block

Responsible for creating a contextualised representation of the input. 

The first step is to create the input embeddings (numerical representation of the 
input tokens): 

1. Break down the input into tokens (using a tokenisation strategy) 
2. Create input embeddings 
    - using a pre-defined vocabulary to map tokens to a numeric representation 
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
