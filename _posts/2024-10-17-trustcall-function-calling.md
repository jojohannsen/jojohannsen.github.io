---
title: "One Function to Do Everything"
date: 2024-10-17
categories: agents
---
Was looking at trustcall yesterday, and what it does for JSON responses from LLMs is a very general solution.  Trustcall automates getting valid data from sloppy LLM responses, fixing things like invalid JSON, hallucinated structure, etc.  

Coding needs similar validation that handles sloppy LLM responses -- did you introduce a new dependency?  Did you change some unrelated code?   Did you just randomly delete some code? 

The hard-code get-off-my-lawn programmer answer to this is "Humans MUST check responses, understand, and take responsibility for it."  

That is a reasonable answer, but that doesn't mean it has to be hard.

What's really requires is **trust** in the answer *before* a human gets involved.  Which is what trustcall does for JSON responses given by a model.

##### JSON Data == Function Call

From the model's point of view, JSON Data and Function Parameters are pretty much the same thing.  The model isn't calling the function, it's just filling in slots in a structure with values, just like it does with JSON Data.

So if you can trust the model with more complex JSON, your function parameters can get really complex.

Why would want such a function?   One case that might make sense is to use such a function during the design/evaluation process.  It's not something people normally do, because it's really unmaintainable and unnecessarily complex.  When the LLM takes that cognitive load with no problem, it's a different story.

Or you might just want to program that way, making a data structure and describing how to process it.

#### The Do-Everything Function

```python
def do_everything_llm(data: EverythingDataStructure, how_to_do_everything: str):
  how_to_do_everything_prompt = f"""
  You are using this data:
  ```json
  {data.as_json()}
  
  Your task is to transform the 'inputs' into the 'outputs',
  using these rules:
  
  {how_to_do_everything}
  
  Please write a python function that will process our data according
  to those rules.   Just put everything into one function, its ok if it's long.
  Let's call it the 'do_everything_function', and have it take an EverythingDataStructure
  as a parameter, and have it also return the updated EverythingDataStructure.
  """
  prompt = ChatPromptTemplate.from_templates(how_to_do_everything_prompt)
  llm = ChatOpenAI()
  chain = prompt | llm
  processing_function = chain.invoke({"data": data, "how_to_do_everything": how_to_do_everything})
  # this defines the 'do_everything_function'
  exec(processing_function)
  everything_result = do_everything_function(data)
  return everything_result
```

