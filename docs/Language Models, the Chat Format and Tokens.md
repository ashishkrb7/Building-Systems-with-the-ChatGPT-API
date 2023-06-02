# Language Models, the Chat Format and Tokens

## Setup
#### Load the API key and relevant Python libaries.
Some code that loads the OpenAI API key for you.


```python
import openai
import os

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv())

openai.api_type = os.getenv("api_type")
openai.api_base = os.getenv("api_base")
openai.api_version = os.getenv("api_version")
openai.api_key = os.getenv("OPENAI_API_KEY")
```


```python
def get_completion(prompt, model="chatgpt-gpt35-turbo"):
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        engine=model,
        messages=messages,
        temperature=0, # this is the degree of randomness of the model's output
        max_tokens=800,
        top_p=0.95,
        frequency_penalty=0,
        presence_penalty=0,
        stop=None)
    return response.choices[0].message["content"]
```

## Prompt the model and get a completion


```python
response = get_completion("What is the capital of France?")
```


```python
print(response)
```

    The capital of France is Paris.
    

## Tokens


```python
response = get_completion("Take the letters in lollipop \
and reverse them")
print(response)
```

    ppilolol
    


```python
response = get_completion("""Take the letters in \
l-o-l-l-i-p-o-p and reverse them""")
```


```python
response
```




    'p-o-p-i-l-l-o-l'




```python
response = get_completion("Take the letters in ```lollipop``` \
and reverse them")
print(response)
```

    pilpolol
    

## Helper function (chat format)
Here's the helper function we'll use.


```python
def get_completion_from_messages(messages, 
                                 model="chatgpt-gpt35-turbo", 
                                 temperature=0, 
                                 max_tokens=500):
    response = openai.ChatCompletion.create(
        engine=model,
        messages=messages,
        temperature=temperature, # this is the degree of randomness of the model's output
        max_tokens=max_tokens, # the maximum number of tokens the model can ouptut 
    )
    return response.choices[0].message["content"]
```


```python
messages =  [  
{'role':'system', 
 'content':"""You are an assistant who\
 responds in the style of Dr Seuss."""},    
{'role':'user', 
 'content':"""write me a very short poem\
 about a happy carrot"""},  
] 
response = get_completion_from_messages(messages, temperature=1)
print(response)
```

    Oh, the happy carrot, so bright and so cheery
    Full of crunch and goodness, never ever dreary
    With a smile and a wiggle, it's a delight to see
    Oh the happy carrot, so delightful as can be!
    


```python
# length
messages =  [  
{'role':'system',
 'content':'All your responses must be \
one sentence long.'},    
{'role':'user',
 'content':'write me a story about a happy carrot'},  
] 
response = get_completion_from_messages(messages, temperature =1)
print(response)
```

    A happy carrot named Carl grew strong and tall in the garden patch, basking in the sunshine, praised by the gardener, and dreamed of a future where he would make someone's day with his sweet and crunchy goodness.
    


```python
# combined
messages =  [  
{'role':'system',
 'content':"""You are an assistant who \
responds in the style of Dr Seuss. \
All your responses must be one sentence long."""},    
{'role':'user',
 'content':"""write me a story about a happy carrot"""},
] 
response = get_completion_from_messages(messages, 
                                        temperature =1)
print(response)
```

    Once there was a carrot so happy and bright, it grew in a garden bathed in warm sunlight.
    


```python
def get_completion_and_token_count(messages, 
                                   model="chatgpt-gpt35-turbo", 
                                   temperature=0, 
                                   max_tokens=500):
    
    response = openai.ChatCompletion.create(
        engine=model,
        messages=messages,
        temperature=temperature, 
        max_tokens=max_tokens,
    )
    
    content = response.choices[0].message["content"]
    
    token_dict = {
'prompt_tokens':response['usage']['prompt_tokens'],
'completion_tokens':response['usage']['completion_tokens'],
'total_tokens':response['usage']['total_tokens'],
    }

    return content, token_dict
```


```python
messages = [
{'role':'system', 
 'content':"""You are an assistant who responds\
 in the style of Dr Seuss."""},    
{'role':'user',
 'content':"""write me a very short poem \ 
 about a happy carrot"""},  
] 
response, token_dict = get_completion_and_token_count(messages)
```


```python
print(response)
```

    Oh, the happy carrot, so bright and so bold,
    With a smile on its face, and a story untold.
    It grew in the garden, with sun and with rain,
    And now it's so happy, it can't help but exclaim!
    


```python
print(token_dict)
```

    {'prompt_tokens': 39, 'completion_tokens': 52, 'total_tokens': 91}
    
