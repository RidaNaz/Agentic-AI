# Hello LangGraph

- LangGraph is an extension of LangChain.
- LangGraph is a library for building stateful, multi-actor applications with LLMs, used to create agent and multi-agent workflows.

- Compared to other LLM frameworks, it offers these core benefits: ***cycles***, ***controllability***, and ***persistence***.

    - LangGraph allows you to define flows that involve ***cycles***, essential for most agentic architectures, differentiating it from DAG-based solutions.
    - As a very low-level framework, it provides fine-grained ***control*** over both the flow and state of your application, crucial for creating reliable agents.
    - Additionally, LangGraph includes built-in ***persistence***, enabling advanced human-in-the-loop and memory features.

### Key Features

- **Cycles and Branching:**  Implement loops and conditionals in your apps.
- **Persistence:**   Automatically save state after each step in the graph. Pause and resume the graph execution at any point to support error recovery, human-in-the-loop workflows, time travel and more.
- **Human-in-the-Loop:**     Interrupt graph execution to approve or edit next action planned by the agent.
- **Streaming Support:**     Stream outputs as they are produced by each node (including token streaming).
- **Integration with LangChain:**    LangGraph integrates seamlessly with LangChain and LangSmith (but does not require them).

### Step by Step Breakdown

1. Initialize the model and tools
2. Initialize graph with state
3. Define graph nodes
4. Define entry point and graph edges
5. Compile the graph
6. Execute the graph

# The Simplest Graph
Let's build a simple graph with 2 nodes and simple edges flow.

![ss](/LangGraph/simplestgraph.png)

## 1. State
- First, define the State of the graph.
- The State schema serves as the input schema for all Nodes and Edges in the graph.
- Let's use the `TypedDict` class from python's typing module as our schema, which provides type hints for the keys.

```py
from typing_extensions import TypedDict

class LearningState(TypedDict):
    prompt: str
```

```py
# prompt: create an example from above LearningState
lahore_state: LearningState = LearningState(prompt= "hello from UMT Lahore")
```

```py
print(lahore_state)
print(lahore_state['prompt'])
print(lahore_state['prompt'] +" I am")
print(lahore_state)
print(type(lahore_state))
```
## 2. Nodes
- Nodes are just python functions.
- The first positional argument is the state, as defined above.
- Because the state is a `TypedDict` with schema as defined above, each node can access the key, `graph_state`, with `state['graph_state']`.
- Each node returns a new value of the state key `graph_state`.
- By default, the new value returned by each node will override the prior state value.

```py
def node_1(state: LearningState) -> LearningState:
    print("---Node 1 State---", state)
    return {"prompt": state['prompt'] +" I am"}

def node_2(state: LearningState) -> LearningState:
    print("---Node 2 State---", state)
    return {"prompt": state['prompt'] +" happy!"}
```

## 3. Edges
- Edges connect the nodes.
- Normal Edges are used if you want to always go from, for example, `node_1` to `node_2`.