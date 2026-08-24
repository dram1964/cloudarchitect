# Large Language Models

LLMs are trained on large text sources and learn everything they
read and also learn the structure and grammar of language. As a 
result they can perform language-based tasks, such as answering 
questions, translating from one language to another.

LLMs are large - their size is measured by the number of parameters 
it has. Parameters are like instructions and rules that tell the model 
how to perform a task. 

LLMs are also trained on huge quantities of text data typically from
internet sources. 

LLMs are general purpose - they are pre-trained on a wide variety of text data 
rather than a specific topic. We can then fine-tune the model to perform 
specific tasks by using domain-specific datasets. 

- Few-shot: minimal data is used to fine-tune a model
- Zero-shot: no data used to fine-tune a model

## Frontier Models

1. GPT - OpenAI
2. Claude - Anthropic
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

At the heart of LLMs lies deep-learning, which uses neural networks
to mimic the functioning of the human brain. 

Deep Learning is a type of model architecture that can excel at finding 
patterns in complex data using a network structure. The network structure
consists of nodes (neurons) that implement a mathematical operation on its
input to produce an output. The strength of the connections (Weights) between
neurons is adjusted during training, to provide the most accurate output by
reducing the difference between the predicted output and the actual output.
Adjustment of Weights is referred to as Optimisation. 

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

## Use Cases

- Content Creation
- Translation
- Answering questions
- Chatbots
- Sentiment Analysis
- Summarisation
- Content Recommendations
- Generating Code
- Medical Diagnosis
- Legal Document Review
- Personalised Marketing

