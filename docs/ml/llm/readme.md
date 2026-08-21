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

Frontier Model: current, state-of-the-art Model.
