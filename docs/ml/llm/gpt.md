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

