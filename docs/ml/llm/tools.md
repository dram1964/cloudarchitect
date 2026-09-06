# Tools

Tools are external functions that are accessible to an LLM. The LLM is 
informed that the tool is available via prompts sent to the LLM. 
The LLM can't execute the tool directly: instead it sends this back as 
a response, and another call needs to be made with the result of the 
tool execution and the original request. 

Tools can be used to: 

- Fetch data or add knowledge
- Carry out actions
- Run code
- Modify the UI

## Agentic AI

There are two ways that an LLM can use tools that form the basis of 
Agentic AI: 

1. Use the tool to call another LLM.
2. Use the tool to track a list of tasks, and record progress towards a goal (Agentic Loop). 

## Add a Tool for OpenAI

First setup OpenAI the openai client:

```python
import os
import json
from dotenv import load_dotenv
from openai import OpenAI
import gradio as gr

load_dotenv(override=True)
openai_api_key = os.getenv('OPENAI_API_KEY')
MODEL = "gpt-4.1-mini"
openai = OpenAI()
```

Setup the system message using the phrase 'If you don't know the answer, then say so.'. This will help avoid AI hallucinations: 

```python
system_message = """
You are a helpful assistant for an Airline called FlightAI.
Give short, courteous answers, no more than 1 sentence.
Always be accurate. If you don't know the answer, say so.
"""
```

Now create a function that will act as the tool to lookup ticket prices: 

```python
# A ticket price tool

ticket_prices = {"london": "$799", "paris": "$899", "tokyo": "$1400", "berlin": "$499"}

def get_ticket_price(destination_city):
    print(f"Tool called for city {destination_city}")
    price = ticket_prices.get(destination_city.lower(), "Unknown ticket price")
    return f"The price of a ticket to {destination_city} is {price}"

```

Create a JSON object in the required OpenAI format, to describe the tool, and
then add this to a list object to be used in the `chat.completions` call: 

```python
# Use JSON to define the tool

price_function = {
    "name": "get_ticket_price",
    "description": "Get the price of a return ticket to the destination city.",
    "parameters": {
        "type": "object",
        "properties": {
            "destination_city": {
                "type": "string",
                "description": "The city that the customer wants to travel to",
            },
        },
        "required": ["destination_city"],
        "additionalProperties": False
    }
}

# Add to the JSON object to the list of tools

tools = [{"type": "function", "function": price_function}]
```

Before calling chat completions, create a function to handle the tool call:

```python
# Write a function to handle a tool call from the LLM

def handle_tool_call(message):
    tool_call = message.tool_calls[0]
    if tool_call.function.name == "get_ticket_price":
        arguments = json.loads(tool_call.function.arguments)
        city = arguments.get('destination_city')
        price_details = get_ticket_price(city)
        response = {
            "role": "tool",
            "content": price_details,
            "tool_call_id": tool_call.id
        }
    return response
```

Now create the chat function and include the `tools` option, adding logic 
to run the tool, and add the response from the tool plus the original prompt
to an additional call to the model:

```python
# Add functionality to the chat.completion to handle the tool call

def chat(message, history):
    messages = [{"role": "system", "content": system_message}] + history + [{"role": "user", "content": message}]
    print(f"Debug: {messages}")
    response = openai.chat.completions.create(
        model=MODEL, 
        messages=messages, 
        tools=tools
    )

    if response.choices[0].finish_reason=="tool_calls":
        message = response.choices[0].message
        response = handle_tool_call(message)
        messages.append(message)
        messages.append(response)
        response = openai.chat.completions.create(model=MODEL, messages=messages)
    
    return response.choices[0].message.content
```

Test the tool call in Gradio:

```python
gr.ChatInterface(fn=chat).launch(inbrowser=True)
```
