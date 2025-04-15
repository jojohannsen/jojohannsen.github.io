---
layout: post
title: "DSL to Code -- mistakes I've made"
date: 2025-04-15
categories: blog
---

I really like langgraph because a lot of times I think of problem workflows in terms of a graph, and langgraph is a language for that.

One problem with langgraph, for me, is that when I code a graph, a lot of times it just doesn't work.   I get errors that don't make any sense to me, because there's some rule or convention I don't know about, and it's never really all that clear what the error is or how to fix it.  Yes, I read the docs, yes, I try examples, and yes, this still happens.  And yes, it's almost always because I've done something stupid.

*While this seems to be my problem, it's not.  It's a bug.  But that's a different post.*

So I made this tool on PyPI, langgraph-codegen, which generates working langgraph code from a specification language.   The language is a made up language, where every word in the language corresponds to a class, function, or field used by langgraph.   The code for each of these is generated, and the graph runs -- so I have a working graph that I can break.  *For me, this is much easier than working for hours trying to come up with a similar starting point.*

So here's an example of the language:

```
# tells jokes, then asks if it should tell another
JokesterState -> get_joke_topic
# first we ask for topic
get_joke_topic -> tell_joke
# then we generate a joke, and display it
tell_joke -> ask_for_another
# then we ask user if they want another joke, we route based on that result
ask_for_another -> tell_another(tell_joke, END)
```

Which breaks down into these langgraph components:

```
Graph Validation Result:
    is_valid: True
    validation_messages: []
    orphan_source_nodes: set()
    orphan_dest_nodes: set()
Graph State:
    state_class_name: JokesterState
    start_field: None
Nodes:
    start_node: get_joke_topic
    source_nodes: {'get_joke_topic', 'ask_for_another', 'tell_joke'}
    dest_nodes: {'END', 'get_joke_topic', 'ask_for_another', 'tell_joke'}
    worker_nodes: set()
Conditional Edges:
    router_functions: {'tell_another'}
```

#### Mistake #1:  not using solveit

The translation from the graph text to the above data structure is deterministic.  So my first strategy was to have an LLM write the code -- it should be able to do this given a description of the graph language, and a prompt.

The resulting code did NOT work.   My description wasn't good enough, or the prompt wasn't good enough.  It almost worked, sometimes it worked perfectly, but it NEVER worked for all my test cases.  *While this did not work, it did cause me to improve my description of the DSL, and my prompt.*

While initially looking great, this approach seemed endless, like I was fighting the LLM just to get one tiny detail right.  The end product is just a function, how hard can it be?

#### Lesson learned: when vibe fails, solveit

So after many hours trying and failing, I did have a clear idea of what I wanted (and was not getting), so I went to solveit, did it piece by piece, and in about an hour had a function that worked perfectly.  

---

### Back to the code

So now I have the graph text, this breakdown of that into langgraph component names, the next task is to generate working langgraph code.

This is the `lgcodegen` program that is part of langgraph-codegen on PyPI.

It runs like this:

`lgcodegen jokester.txt --code`

which generates the langgraph code, and makes a runnable graph.   The resulting graph can be run:

`python jokester/jokester.py`

This I've been coding with different tools -- Claude Code, Aider, Cursor, solveit.  

Here's what this looks like:
```
(py312) johannesjohannsen@Johanness-MacBook-Pro evals % lgcodegen jokester.txt --code
LangGraph CodeGen v1.0.2
Graph source: jokester.txt
Creating folder jokester
Graph specification: jokester/jokester.txt
To regenerate python cod from graph spec: lgcodegen jokester/jokester.txt --code
To run:  python jokester/jokester.py
(py312) johannesjohannsen@Johanness-MacBook-Pro evals % python jokester/jokester.py
NODE: get_joke_topic

    {'get_joke_topic': {'nodes_visited': 'get_joke_topic', 'counter': 1}}

NODE: tell_joke

    {'tell_joke': {'nodes_visited': 'tell_joke', 'counter': 2}}

NODE: ask_for_another
CONDITION: tell_another_tell_joke. Result: True

    {'ask_for_another': {'nodes_visited': 'ask_for_another', 'counter': 3}}

NODE: tell_joke

    {'tell_joke': {'nodes_visited': 'tell_joke', 'counter': 4}}

NODE: ask_for_another
CONDITION: tell_another_tell_joke. Result: True

    {'ask_for_another': {'nodes_visited': 'ask_for_another', 'counter': 5}}

NODE: tell_joke

    {'tell_joke': {'nodes_visited': 'tell_joke', 'counter': 6}}

NODE: ask_for_another
CONDITION: tell_another_tell_joke. Result: True

    {'ask_for_another': {'nodes_visited': 'ask_for_another', 'counter': 7}}

NODE: tell_joke

    {'tell_joke': {'nodes_visited': 'tell_joke', 'counter': 8}}

NODE: ask_for_another
CONDITION: tell_another_tell_joke. Result: False
CONDITION: tell_another_END. Result: True

    {'ask_for_another': {'nodes_visited': 'ask_for_another', 'counter': 9}}

DONE STREAMING, final state:
StateSnapshot(values={'nodes_visited': ['get_joke_topic', 'tell_joke', 'ask_for_another', 'tell_joke', 'ask_for_another', 'tell_joke', 'ask_for_another', 'tell_joke', 'ask_for_another'], 'counter': 9}, next=(), config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1f019f32-31e0-6150-8009-44763177c2be'}}, metadata={'source': 'loop', 'writes': {'ask_for_another': {'nodes_visited': 'ask_for_another', 'counter': 9}}, 'step': 9, 'parents': {}, 'thread_id': '1'}, created_at='2025-04-15T12:14:10.540166+00:00', parent_config={'configurable': {'thread_id': '1', 'checkpoint_ns': '', 'checkpoint_id': '1f019f32-31de-69cc-8008-764e8feb5569'}}, tasks=())
```

##### This is NOT very useful

While `lgcodegen` does a decent job creating the code, and it runs, it's not that great a starting point.   The main problem is the state is NOT the one the graph needs.   Since every function takes that state as a parameter, all that code has to change once you introduce a more realistic state.

#### Mistake #2:  not using prompt chain

This guy IndyDevDan on youtube has the best explanations of how to use prompt chains, the main takeaway for me was to create a workflow that used LLMs to write the langgraph code.

`lgcodegen` does not use this -- it's something I've done in same repo, but in standalone functions and a script that runs them in the correct order.

*It makes no sense to generate code without using an LLM -- the output is unrealistic, limited, and inflexible.  Chaining prompts with LLM is simpler and gives more useful results.*

#### Lesson learned: the amount of compute applied to problem matters

See IndyDevDan for details on this.

---

The LLM inclusive approach makes 3 markdown specifications for graph components, and 3 python files.  

This results in 6 different python programs:

- mk_state_spec.py
- mk_state_code.py
- mk_node_spec.py
- mk_node_code.py
- mk_graph_spec.py
- mk_graph_code.py

These have very little and very similar code.   All they do is load context, grab a prompt, and run an LLM agent to execute it.  The agent writes the file.

*This code is **much** simpler than the `lgcodegen` code, and the end result is **much** better*.

The end result is a graph where the State and behavior of the Node functions is more realistic -- LLMs are invoked, human input is handled -- it's much closer to the graph you want.  

##### Generating and Running the graph

Here's what the generated graph run looks like -- the generated code includes prompts and LLM calls and human input, and realistic State.  Those lines: `Would you like to hear another joke?` really are waiting for human input.  The generated jokes were from an LLM call.

```bash
(py312) johannesjohannsen@Johanness-MacBook-Pro evals % ./run_eval.sh
Available .txt files:
1. evaluator_optimizer.txt
2. handle_line.txt
3. jokester.txt
4. lgcodegen.txt
5. md_consensus.txt
6. orchestrator_worker.txt
7. parallelization.txt
8. prompt_chaining.txt
9. routing.txt
10. x.txt
Enter number of file to process: 3
Processing jokester.txt...
+ rm -rf jokester
+ python mk_state_spec.py jokester.txt
Creating: jokester/jokester.yaml
Reading: jokester/jokester.yaml
Graph folder: jokester
Agent Model: gpt-4.1
Reading prompt from file: prompts/graph-spec.md
INFO Saved: jokester/state_spec.md
Python files: []
+ python mk_state_code.py jokester.txt
Reading: jokester/jokester.yaml
Graph folder: jokester
Agent Model: gpt-4.1
State spec: jokester/state_spec.md
Reading prompt from file: prompts/graph-spec.md
INFO Saved: jokester/state_code.py
Successfully generated: jokester/state_code.py
+ python mk_node_spec.py jokester.txt
Reading: jokester/jokester.yaml
Graph folder: jokester
Agent Model: gpt-4.1
State spec file: jokester/state_spec.md
Node spec file does not exist: jokester/node_spec.md
Node spec file: jokester/node_spec.md
Reading prompt from file: prompts/graph-spec.md
INFO Saved: jokester/node_spec.md
Successfully generated: jokester/node_spec.md
+ python mk_node_code.py jokester.txt
Reading: jokester/jokester.yaml
Graph folder: jokester
Agent Model: gpt-4.1
State spec file: jokester/state_spec.md
Reading prompt from file: prompts/graph-spec.md
Reading prompt from file: prompts/human-questionary-input.md
Reading prompt from file: prompts/node-code-example.md
INFO Saved: jokester/node_code.py
+ python mk_graph_spec.py jokester.txt
Reading: jokester/jokester.yaml
Graph folder: jokester
Agent Model: gpt-4.1
State spec file: jokester/state_spec.md
Reading prompt from file: prompts/graph-spec.md
INFO Saved: jokester/graph_spec.md
Graph spec file: jokester/graph_spec.md
+ python mk_graph_code.py jokester.txt
Reading: jokester/jokester.yaml
Graph folder: jokester
Agent Model: gpt-4.1
State spec file: jokester/state_spec.md
Reading prompt from file: prompts/graph-spec.md
INFO Saved: jokester/graph_code.py
Graph code file: jokester/graph_code.py
(py312) johannesjohannsen@Johanness-MacBook-Pro evals % python jokester/graph_code.py
? Please provide a topic you'd like to hear a joke about. flowers

    {'get_joke_topic': {'joke_topic': 'flowers'}}


    {'tell_joke': {'joke_text': "Here's a flower joke for you:\n\nWhy don't flowers ride bikes?\nBecause they lost their petals (pedals)! 🌸\n\nHere's another one:\nWhat did the bee say to the flower?\nHey honey, what's buzzin'? 🐝🌺"}}

? Would you like to hear another joke? Yes

    {'ask_for_another': {'wants_another': True}}


    {'tell_joke': {'joke_text': "Here's a flower joke for you:\n\nWhat did the bee say to the flower?\nHey bud, when do you plan to bloom? I'm running out of pollen time!\n\nAlternative flower joke:\nWhat's a flower's favorite kind of pickle?\nA daisy dill!"}}

? Would you like to hear another joke? No

    {'ask_for_another': {'wants_another': False}}

DONE STREAMING, final state:
  joke_topic: flowers
  joke_text: Here's a flower joke for you:

What did the bee say to the flower?
Hey bud, when do you plan to bloom? I'm running out of pollen time!

Alternative flower joke:
What's a flower's favorite kind of pickle?
A daisy dill!
  wants_another: False
```

A few things to note about this:

1. The python programs have NOTHING about what code or markdown to generate -- that is all in the prompts.  Using langsmith hub for this made prompt development much simpler, the program itself never changed.

2. The python programs are all very similar:

   - Load context (prompts, previously generated files)
   - Prompt describes how to generate the resulting file
   - Get model to use from config, execute the prompt, check for resulting file

3. The end result is realistic, and is a surprising amount of code given the very limited specification:
   ```
   # tells jokes, then asks if it should tell another
   JokesterState -> get_joke_topic
   # first we ask for topic
   get_joke_topic -> tell_joke
   # then we generate a joke, and display it
   tell_joke -> ask_for_another
   # then we ask user if they want another joke, we route based on that result
   ask_for_another -> tell_another(tell_joke, END)
   ```

   From *only the above*, we get the following code files:

##### State Code

```python
# This file was generated by gpt-4.1
from typing import Optional
from pydantic import BaseModel, Field

class JokesterState(BaseModel):
    joke_topic: Optional[str] = Field(default=None, description="The topic about which a joke will be generated. Typically input provided by the user and forms the basis for joke generation.")
    joke_text: Optional[str] = Field(default=None, description="The actual joke generated based on the topic. It is displayed to the user after generation.")
    wants_another: Optional[bool] = Field(default=None, description="Whether the user would like to hear another joke. Used for routing control, as determined by the user's response.")
```

##### Node Code

```python
import questionary
from langchain_anthropic import ChatAnthropic
from state_code import JokesterState
from langchain_core.runnables.config import RunnableConfig

# Node: get_joke_topic
def get_joke_topic(state: JokesterState, *, config: RunnableConfig = None):
    """Ask the user for a topic for the joke and write it to 'joke_topic'."""
    topic = questionary.text("Please provide a topic you'd like to hear a joke about.").ask()
    return {"joke_topic": topic}

# Node: tell_joke
def tell_joke(state: JokesterState, *, config: RunnableConfig = None):
    """Generate a joke about the topic stored in 'joke_topic' and write it to 'joke_text'."""
    llm = ChatAnthropic(model="claude-3-5-sonnet-latest")
    prompt = f"Generate a joke about the topic: {state.joke_topic}"
    msg = llm.invoke(prompt)
    return {"joke_text": msg.content}

# Node: ask_for_another
def ask_for_another(state: JokesterState, *, config: RunnableConfig = None):
    """Ask the user if they want another joke and write result to 'wants_another'."""
    answer = questionary.confirm("Would you like to hear another joke?").ask()
    return {"wants_another": answer}
```

##### Langgraph Code

```python
# This file was generated by gpt-4.1
from langgraph.graph import START, END, StateGraph
from langgraph.checkpoint.memory import MemorySaver
from state_code import JokesterState
from node_code import get_joke_topic, tell_joke, ask_for_another
from langchain_core.runnables.config import RunnableConfig

# Routing Function for 'ask_for_another' node
def tell_another(state: JokesterState):
    # Decide whether to route to 'tell_joke' or END based on wants_another
    if getattr(state, 'wants_another', None):
        return 'tell_joke'
    return 'END'

def build_jokester_graph():
    checkpoint_saver = MemorySaver()
    builder_jokester = StateGraph(JokesterState)
    
    builder_jokester.add_node('get_joke_topic', get_joke_topic)
    builder_jokester.add_node('tell_joke', tell_joke)
    builder_jokester.add_node('ask_for_another', ask_for_another)

    builder_jokester.add_edge(START, 'get_joke_topic')
    builder_jokester.add_edge('get_joke_topic', 'tell_joke')
    builder_jokester.add_edge('tell_joke', 'ask_for_another')

    # Setup routing
    routing_edges = {'tell_joke': 'tell_joke', 'END': END}
    builder_jokester.add_conditional_edges('ask_for_another', tell_another, routing_edges)
    
    return builder_jokester.compile(checkpointer=checkpoint_saver)

# Instantiate compiled graph for main section
jokester_graph = build_jokester_graph()

if __name__ == "__main__":
    # Create the graph
    workflow = jokester_graph
    config = RunnableConfig(configurable={"thread_id": "1"})
    for output in workflow.stream(JokesterState(), config=config):
        print(f"\n    {output}\n")
    print("DONE STREAMING, final state:")
    state = workflow.get_state(config)
    for k,v in state.values.items():
        print(f"  {k}: {v}")
```

#### 
