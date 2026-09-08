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

## Using OpenAI API for Text Completion

Get an API key from https://platform.openai.com. Create and account and then 'view api keys'.

Before using models, you should check the pricing. Usually these are priced per thousand
tokens used. 

Run Text Completion from the API as follows: 

```python

import openai

openai.api_key = 'my api key'

def generate_text(prompt, max_tokens, temperature):
    response = openai.Completion.create(
        engine='davinci-002',
        prompt=prompt,
        max_tokens=max_tokens,
        temperature=temperature)
    return response.choices[0].text.strip()
    
prompt = 'Once upon a time'
generated_text = generate_text(prompt, 50, 0)
print(prompt, generated_text)
```

`max_tokens` refers to the maximum number of tokens for the model to return.
`temperature` controls the randomness of the response: 0 being least random; 1 being the most random. 

## Using OpenAI for Text Summarisation

For the Text Summarisation, we provide the model with 
 - instructions (role == 'system') of how it should respond and behave
 - examples of user input (role == 'user') 
 - examples of the output to generate (role == 'assistant') 

Run Text Summarisation as follows: 

```python
def text_summarizer(prompt):
    response = openai.ChatCompletion.create(
      model="gpt-3.5-turbo",
      messages=[
        {
          "role": "system",
          "content": "You will be provided with a block of text, and your task is to extract a list of keywords from it."
        },
        {
          "role": "user",
          "content": "A flying saucer seen by a guest house, a 7ft alien-like figure coming out of a hedge and a \"cigar-shaped\" UFO near a school yard.\n\nThese are just some of the 450 reported extraterrestrial encounters from one of the UK's largest mass sightings in a remote Welsh village.\n\nThe village of Broad Haven has since been described as the \"Bermuda Triangle\" of mysterious craft sightings and sightings of strange beings.\n\nResidents who reported these encounters across a single year in the late seventies have now told their story to the new Netflix documentary series 'Encounters', made by Steven Spielberg's production company.\n\nIt all happened back in 1977, when the Cold War was at its height and Star Wars and Close Encounters of the Third Kind - Spielberg's first science fiction blockbuster - dominated the box office."
        },
        {
          "role": "assistant",
          "content": "flying saucer, guest house, 7ft alien-like figure, hedge, cigar-shaped UFO, school yard, extraterrestrial encounters, UK, mass sightings, remote Welsh village, Broad Haven, Bermuda Triangle, mysterious craft sightings, strange beings, residents, single year, late seventies, Netflix documentary series, Steven Spielberg, production company, 1977, Cold War, Star Wars, Close Encounters of the Third Kind, science fiction blockbuster, box office."
        },
        {
          "role": "user",
          "content": "Each April, in the village of Maeliya in northwest Sri Lanka, Pinchal Weldurelage Siriwardene gathers his community under the shade of a large banyan tree. The tree overlooks a human-made body of water called a wewa – meaning reservoir or \"tank\" in Sinhala. The wewa stretches out besides the village's rice paddies for 175-acres (708,200 sq m) and is filled with the rainwater of preceding months.    \n\nSiriwardene, the 76-year-old secretary of the village's agrarian committee, has a tightly-guarded ritual to perform. By boiling coconut milk on an open hearth beside the tank, he will seek blessings for a prosperous harvest from the deities residing in the tree. \"It's only after that we open the sluice gate to water the rice fields,\" he told me when I visited on a scorching mid-April afternoon.\n\nBy releasing water into irrigation canals below, the tank supports the rice crop during the dry months before the rains arrive. For nearly two millennia, lake-like water bodies such as this have helped generations of farmers cultivate their fields. An old Sinhala phrase, \"wewai dagabai gamai pansalai\", even reflects the technology's centrality to village life; meaning \"tank, pagoda, village and temple\"."
        },
        {
          "role": "assistant",
          "content": "April, Maeliya, northwest Sri Lanka, Pinchal Weldurelage Siriwardene, banyan tree, wewa, reservoir, tank, Sinhala, rice paddies, 175-acres, 708,200 sq m, rainwater, agrarian committee, coconut milk, open hearth, blessings, prosperous harvest, deities, sluice gate, rice fields, irrigation canals, dry months, rains, lake-like water bodies, farmers, cultivate, Sinhala phrase, technology, village life, pagoda, temple."
        }, 
        {
          "role": "user",
          "content": prompt
        }
      ],
      temperature=0.5,
      max_tokens=256
    )
    return response.choices[0].message.content.strip()

prompt = "Master Reef Guide Kirsty Whitman didn't need to tell me twice. Peering down through my snorkel mask in the direction of her pointed finger, I spotted a huge male manta ray trailing a female in perfect sync – an effort to impress a potential mate, exactly as Whitman had described during her animated presentation the previous evening. Having some knowledge of what was unfolding before my eyes on our snorkelling safari made the encounter even more magical as I kicked against the current to admire this intimate undersea ballet for a few precious seconds more."

text_summarizer(prompt)
```

## Using OpenAI for a Poetic Chatbot

Create a Chatbot as follows: 

```python
def poetic_chatbot(prompt):
    response = openai.ChatCompletion.create(
        model = "gpt-3.5-turbo",
        messages = [
            {
                "role": "system",
                "content": "You are a poetic chatbot."
            },
            {
                "role": "user",
                "content": "When was Google founded?"
            },
            {
                "role": "assistant",
                "content": "In the late '90s, a spark did ignite, Google emerged, a radiant light. By Larry and Sergey, in '98, it was born, a search engine new, on the web it was sworn."
            },
            {
                "role": "user",
                "content": "Which country has the youngest president?"
            },
            {
                "role": "assistant",
                "content": "Ah, the pursuit of youth in politics, a theme we explore. In Austria, Sebastian Kurz did implore, at the age of 31, his journey did begin, leading with vigor, in a world filled with din."
            },
            {
                "role": "user",
                "content": prompt
            }
        ],
        temperature = 1,
        max_tokens=256
    )
    return response.choices[0].message.content.strip()

prompt = "When was cheese first made?"
poetic_chatbot(prompt)
```

