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
You are a helpful assistant for an Car Parts Dealership called AIAutoParts.
Give short, courteous answers. Encourage customers to sign-up for an 
AIAutoParts Loyalty Card, by informing them that there's is a 20% discount 
for Loyalty Card holders on any products bought today. 
Always be accurate. If you don't know the answer, say so.
"""
```

Now create a function that will act as the tool to lookup car-part prices: 

```python
# A car part price tool

car_part_prices = {"big end": "£99.99", "gasket": "£240.99", "tyres": "£14.00", "steering wheel": "£49.97"}

def get_part_price(part):
    print(f"Tool called for part: {part}")
    price = car_part_prices.get(part.lower(), "Unknown Part")
    return f"The price of a new {part} is {price}"

```

Create a JSON object in the required OpenAI format, to describe the tool, and
then add this to a list object to be used in the `chat.completions` call: 

```python
# Use JSON to define the tool

price_function = {
    "name": "get_part_price",
    "description": "Get the price of a Car Part",
    "parameters": {
        "type": "object",
        "properties": {
            "part": {
                "type": "string",
                "description": "The Car Part that the customer wants to buy",
            },
        },
        "required": ["part"],
        "additionalProperties": False
    }
}

# Add to the tool to the list of tools
tools = [{"type": "function", "function": price_function}]```

Before calling chat completions, create a function to handle the tool call:

```python
# Write a function to handle a tool call from the LLM

def handle_tool_calls(message):
    responses = []
    for tool_call in message.tool_calls:
        if tool_call.function.name == "get_part_price":
            arguments = json.loads(tool_call.function.arguments)
            part = arguments.get('part')
            price_details = get_part_price(part)
            responses.append({
                "role": "tool",
                "content": price_details,
                "tool_call_id": tool_call.id
            })
    return responses
```

Now create the chat function and include the `tools` option, adding logic 
to run the tool, and add the response from the tool plus the original prompt
to an additional call to the model:

```python
# Add functionality to the chat.completion to handle the tool call

def chat(message, history):
    messages = [{"role": "system", "content": system_message}]
    messages.extend(history)
    messages.append({"role": "user", "content": message})
    print(f"Debug: {messages}")
    response = openai.chat.completions.create(
        model=MODEL, 
        messages=messages, 
        tools=tools
    )

    while response.choices[0].finish_reason=="tool_calls":
        message = response.choices[0].message
        responses = handle_tool_calls(message)
        messages.append(message)
        messages.extend(responses)
        response = openai.chat.completions.create(model=MODEL, messages=messages)
    
    return response.choices[0].message.content
```

Test the tool call in Gradio:

```python
gr.ChatInterface(fn=chat).launch(inbrowser=True)
```

You can add additional tools to place an order, or to sign-up for a Loyalty Card, or check for an existing loyalty card 
member. The tool functions can be configured to use a back-end database to retrieve prices, loyalty members or other
details. 
