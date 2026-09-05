# Ollama

Ollama allows you to run open-source models on your own computer. You can
run a model with: 

```bash
ollama run gemma4
```

To run the same model in the cloud use: 

```bash
ollama run gemma4:cloud
```

Exit your chat with `/bye`.

Use `ollama list` to list the models installed locally. These are typically stored 
in `/usr/share/ollama/.ollama/models` on an Ubuntu system. Remove models with 
`ollama rm` 

List running models with `ollama ps` and use `ollama stop` to stop a particular model.

If you have an Nvidia GPU, check that you have CUDA installed by running `nvidia-smi`. 

Instructions to [configure ollama as a system service](https://docs.ollama.com/linux#adding-ollama-as-a-startup-service-recommended)

## Environment Setup

1. Clone the Repo. Install Cursor. Open the Project.
2. Install uv. Setup your environment. 
3. Create an OpenAI key
4. Create the `.env` file

## Glossary

- Frontier Model: current, state-of-the-art Models create by large companies. 
Tend to be closed-source. 
- Open-Source Model: really open-weight model, because the weights are publicly
exposed, but the training data and methodologies are not. 
- GPT : Generative Pre-Trained Transformer
- Distillation: using one LLM to create data for training another smaller LLM
- Inference: sending inputs to a model and getting outputs
- Agentic AI: works like a digital assistant to perform tasks. Agents are LLMs
that control the workflow, e.g. by calling other LLMs or using tools to perform
a task. Agentic AI is often described as autonomous AI: the Agentic AI decides
how it will complete the task and generates its own plan. 
- RLHF: Reinforcement Learning from Human Feedback
- LSTM: Long Short Term Memory - architecture used pre-transformers. Difficult to run this architecture in parallel, as each step relies on the output from the previous. 
- Emergent Intelligence: refers to the fact that LLMs are capable of not only
producing plausible answers, but also accurate answers. 
- Prompt Engineering: the skill to know how to best provide prompts to LLMs to 
produce better responses
- Context Engineering: as prompt engineering skills become more widespread, 
context engineering became the next key-skill for optimising LLMs. Involves
providing the correct domain-specific tokens for optimising responses. 
- Context Window: the maximum number of tokens an LLM can consider when generating the next token
- Multi-shot prompting: where you provide multiple inputs including example prompts and responses
- Prompt Caching: can be used to reduce spend by caching some of the prompt tokens
