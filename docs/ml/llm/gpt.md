# ChatGPT

GPT stands for 'Generative Pre-Trained Transformer'. ChatGPT Published by OpenAI: 

- GPT-1. Released 2018, 117 million parameters, 40GB of training text from internet
- GPT-2. Released 2019, 1.5 billion parameters
- GPT-3. Released 2020, 175 billion parameters
- GPT-3.5. Released 2022. Three variants 1.3B, 6B and 175B parameters. Designed to elimate 
    toxic outputs using reinforcement learning with human feedback (RLHF).  
- GPT-4. Released 2023, 1 trillion parameters
- GPT-5. Released 2025. The first reasoninig model. Because of the overhead
of 'reasoning', GPT-4 can often out-perform GPT-5 in chat-based scenarios. 

Optimised for conversational interactions. 

OpenAI provide an API to interact with these models

[OpenAI Playground](https://platform.openai.com/playground) to test out different models and
settings. 

## Using OpenAI Models

To interact with the models available on OpenAI with Python use the `openai` 
module: 

```python
import os
import dotenv
import openai

load_dotenv()
api_key = os.getenv('OPENAI_API_KEY')

model = 'gpt-5.6-sol'

system_prompt="""
# Instructions for the model go here
"""

user_prompt="""
# Questions from the user go here
"""

messages = [
    {"role": system, "content": system_prompt},
    {"role": user, "content": user_prompt},
]

response = openai.chat.completions.create(
    model = model
    messages = messages

print(response.choices[0].message.content
```

Message content can be constructed using functions, e.g. to scrape a website or read a document. 

## OpenAI API

The `openai` Python library is a light-weight wrapper to generate the http
POST requests to the OpenAI API endpoint: 

```python
import requests
import os
from dotenv import load_dotenv

load_dotenv(override=True)
api_key = os.getenv('OPENAI_API_KEY')

headers = {"Authorization": f"Bearer {api_key}", "Content-Type": "application/json"}

payload = {
    "model": "gpt-5-nano",
    "messages": [
        {"role": "user", 
        "content": "Tell me a fun fact"}
    ]
}

response = requests.post(
    "https://api.openai.com/v1/chat/completions",
    headers=headers,
    json=payload
)

response.json()["choices"][0]["message"]["content"]
```

OpenAI Chat Completions API is now the *de facto standard* API and was adopted
by other vendors to provide an OpenAI-compatible API. 

Because other vendors adopted this API standard, OpenAI developed the 
`openai` library to accept an Endpoint URL and api_key parameter to 
allow usage on compatible APIs from other vendors: 

```python
GEMINI_BASE_URL = "https://generativelanguage.googleapis.com/v1beta/openai/"
load_dotenv(override=True)
google_api_key = os.getenv("GOOGLE_API_KEY")

gemini = OpenAI(base_url=GEMINI_BASE_URL, api_key=google_api_key)

response = gemini.chat.completions.create(
    model="gemini-3.1-flash-lite", 
    messages=[
        { "role": "user", 
        "content": "Tell me a fun fact" 
        }
    ]
)

response.choices[0].message.content
```

And you can use the same method to connect to Ollama running locally on 
your device: 

```python
OLLAMA_BASE_URL = "http://localhost:11434/v1"
ollama = OpenAI(base_url=OLLAMA_BASE_URL, api_key='ollama')

response = ollama.chat.completions.create(
    model="llama3.2", 
    messages=[
        {"role": "user", 
        "content": "Tell me a fun fact"
        }
    ]
)

response.choices[0].message.content
```

## API Keys and Base URLs

API Keys for each platform typically come with an upfront cost-commitment. 
You can sign-up for the various plaforms using the URLs below: 

1. OpenAI at https://openai.com/api/  
2. Anthropic at https://console.anthropic.com/  
3. Google at https://aistudio.google.com/   
4. DeepSeek at  https://platform.deepseek.com/  
5. Groq at https://console.groq.com/  
6. Grok at https://console.x.ai/  

OpenRouter is an alternative, unified interface to other providers models at
https://openrouter.ai/. 

Setup clients for each API by specifying the API Key to use and the Base URL 
for each platform: 

```python
openai = OpenAI()

anthropic_url = "https://api.anthropic.com/v1/"
gemini_url = "https://generativelanguage.googleapis.com/v1beta/openai/"
deepseek_url = "https://api.deepseek.com"
groq_url = "https://api.groq.com/openai/v1"
grok_url = "https://api.x.ai/v1"
openrouter_url = "https://openrouter.ai/api/v1"
ollama_url = "http://localhost:11434/v1"

anthropic = OpenAI(api_key=anthropic_api_key, base_url=anthropic_url)
gemini = OpenAI(api_key=google_api_key, base_url=gemini_url)
deepseek = OpenAI(api_key=deepseek_api_key, base_url=deepseek_url)
groq = OpenAI(api_key=groq_api_key, base_url=groq_url)
grok = OpenAI(api_key=grok_api_key, base_url=grok_url)
openrouter = OpenAI(base_url=openrouter_url, api_key=openrouter_api_key)
ollama = OpenAI(base_url=ollama_url, api_key="ollama")
```

## Platform Specific APIs

Although, most platforms provide OpenAI-compatible APIs, they still 
retain their vendor specific APIs. These are documented on their platform
pages. 

To use the anthropic API, try: 

```python
import anthropic

# uses ANTHROPIC_API_KEY or credentials from `ant auth login`
client = anthropic.Anthropic()

with client.messages.stream(
    model="claude-sonnet-5",
    max_tokens=20000,
    messages=[],
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

For Gemini, try: 

```python
from google import genai

# Uses GEMINI_API_KEY
client = genai.Client()

interaction = client.interactions.create(
    model="gemini-3.7-flash",
    input="Explain how AI works in a few words"
)
print(interaction.output_text)
```

## LangChain and LiteLLM

Both LangChain and LiteLLM can be used to make API calls. For LangChain try:

```python
import os
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from IPython.display import Markdown, display

load_dotenv()
api_key = os.getenv('OPENAI_API_KEY')


llm = ChatOpenAI(model="gpt-5-mini")

prompt = [{
    'role': 'user', 'content': 'Explain how AI works in a few words'
}]

response = llm.invoke(prompt)

display(Markdown(response.content))
```

For LiteLLM try: 

```python
from litellm import completion
from IPython.display import Markdown, display

prompt = [{
    'role': 'user', 'content': 'Explain how Machine Learning works'
}]

# uses OPENAI_API_KEY in the environment
response = completion(model="openai/gpt-4.1", messages=prompt)
reply = response.choices[0].message.content
display(Markdown(reply))
```

LiteLLM also provides access to data about Token usage and Costs for each
call: 

```python
print(f"Input tokens: {response.usage.prompt_tokens}")
print(f"Output tokens: {response.usage.completion_tokens}")
print(f"Total tokens: {response.usage.total_tokens}")
print(f"Total cost: {response._hidden_params["response_cost"]*100:.4f} cents")
```

## Streaming the Response

You can also stream the response from `openai`: 

```python
# imports
import os
from dotenv import load_dotenv
import openai
from IPython.display import Markdown, display, update_display

# set up environment
load_dotenv(override=True)
openai.api_key = os.getenv('OPENAI_API_KEY')

# set up prompts
question = """
Please explain what this code does and why:
yield from {book.get("author") for book in books if book.get("author")}
"""

system_prompt = """
You are an expert in all fields related to information technology. 
You are able to interpret complicated code and provide clear and concise
analysis in plain English. If the question is about a block of code, 
provide one or more worked examples of how you would use this code. 
"""

# add streaming function
def stream_response(system_prompt, user_prompt):
    stream = openai.chat.completions.create(
        model="gpt-4.1-mini",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt}
          ],
        stream=True
    )    
    response = ""
    display_handle = display(Markdown(""), display_id=True)
    for chunk in stream:
        response += chunk.choices[0].delta.content or ''
        update_display(Markdown(response), display_id=display_handle.display_id)

# stream response
stream_response(system_prompt, question)

```

## Getting a JSON Response

You can prompt the LLM to return JSON: 

```python
# imports
...
import json

system_prompt = """
You will be provided with a list of Books in CSV format. You should respond
with a list of book authors and titles in JSON format as in this example:

{
    "books": [
        {"author": "Charles Dickens", "title": "A Tale of Two Cities"},
        {"author": "J.D. Salinger", "title": "Catcher in the Rye"}
    ]
}
"""

def get_csv_rows(file):
    user_prompt = """
Here is the list of Books with Authors and Titles included:

"""

    csv_list = """
# csv data goes here
"""

    user_prompt += csv_list
    return user_prompt

def select_book_authors(file):
    response = openai.chat.completions.create(
        model=MODEL,
        messages=[
            {"role": "system", "content": link_system_prompt},
            {"role": "user", "content": get_csv_rows(file)}
        ],
        response_format={"type": "json_object"}
    )
    result = response.choices[0].message.content
    books = json.loads(result)
    return result

select_book_authors('./data/books.csv')
```

## Image Generation

You can use `openai.images.generate` to create a new image:

```python
import os
from dotenv import load_dotenv
from openai import OpenAI
import base64
from io import BytesIO
from PIL import Image

load_dotenv(override=True)
openai_api_key = os.getenv('OPENAI_API_KEY') 
openai = OpenAI()

MODEL = "gpt-image-1-mini"

def artist(prompt):
    image_response = openai.images.generate(
            model=MODEL,
            prompt=prompt,
            size="1024x768",
            n=1,
        )
    image_base64 = image_response.data[0].b64_json
    image_data = base64.b64decode(image_base64)
    return Image.open(BytesIO(image_data))

artist_prompt="An image representing Christmas 2026, showing typical symbols of Christmas in the style of Edvard Munch"
filename="munch_christmas_2026.png"

image = artist(artist_prompt)
display(image)

image.save(f"./images/{filename}")
```

You can also use the `openai.images.edit` to create a new image from 
one or more existing input images: 

```python
import os
import base64
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv(override=True)
openai_api_key = os.getenv('OPENAI_API_KEY') 
client = OpenAI()

prompt = """
Generate an artistic version of the input image in the style of Gaughin
"""

result = client.images.edit(
    model="gpt-image-2",
    image=[
        open("input/stratford_tapas.jpg", "rb"),
    ],
    prompt=prompt,
)

image_base64 = result.data[0].b64_json
image_bytes = base64.b64decode(image_base64)

# Save the image to a file
with open("output/gaughin_stratford.png", "wb") as f:
    f.write(image_bytes)
```

See [Image Generation](https://developers.openai.com/api/docs/guides/image-generation?mask-edit-api=image#overview) for more information on using 
`chat.images` or `chat.responses` to create or edit images. 

## Using OpenAI for Speech Generation

Use `openai.audio.speech` to generate audio from input text:


```python
import os
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv(override=True)
openai_api_key = os.getenv('OPENAI_API_KEY') 

client = OpenAI()
speech_file_path = "audio/speech.mp3"

with client.audio.speech.with_streaming_response.create(
    model="gpt-4o-mini-tts",
    voice="coral",
    input="Now is the winter of our discontent made glorious summer by this noble son of York.",
    instructions="Speak in a cheerful and positive tone.",
) as response:
    response.stream_to_file(speech_file_path)
```

See [Text to Speech](https://developers.openai.com/api/docs/guides/text-to-speech) for more info. 

