# Graph Construction

- Now, we build the graph from our components defined in [RAEDME.md](/LangGraph/README.md#the-simplest-graph).
- The `StateGraph` class is the graph class that we can use.
- First, we initialize a `StateGraph` with the `State` class we defined above.
- Then, we add our nodes and edges.
- We use the `START` Node, a special node that sends user input to the graph, to indicate where to start our graph.
- The `END` Node is a special node that represents a terminal node.
- Finally, we compile our graph to perform a few basic checks on the graph structure.
- We can visualize the graph as a Mermaid diagram.

```py
from IPython.display import Image, display # Preview Graph

from langgraph.graph import StateGraph, START, END
from langgraph.graph.state import CompiledStateGraph # type

# Build graph
builder: StateGraph = StateGraph(state_schema=LearningState)
```

```py
# Nodes
builder.add_node("node_1", node_1)
builder.add_node("node_2", node_2)
```

```py
# Simples Edges Logic
builder.add_edge(START, "node_1")
builder.add_edge("node_1", "node_2")
builder.add_edge("node_2", END)
```

```py
# Add
graph: CompiledStateGraph = builder.compile()
```

```py
print(graph)
```

```py
print(graph.get_graph())
```

```py
# View
display(Image(graph.get_graph().draw_mermaid_png()))
```

![ss](/LangGraph/simplestgraph2.jpeg)

# Graph Invocation

```py
graph.invoke({"prompt" : "Hi"})
```

- `invoke` is one of the standard methods in this interface and it runs the entire graph synchronously.
- It returns the final state of the graph after all nodes have executed.

# In Nodes Use LLM == GoogleChatModel in Langchain

```py
%pip install -q -U langchain
%pip install -q -U langchain-google-genai
```

```py
from google.colab import userdata
google_api_key = userdata.get('GEMINI_API_KEY')
```

```py
from langchain_google_genai import ChatGoogleGenerativeAI

# Initialize an instance of the ChatGoogleGenerativeAI with specific parameters
llm: ChatGoogleGenerativeAI = ChatGoogleGenerativeAI(
    model="gemini-1.5-flash",  # Specify the model to use
    api_key=google_api_key,     # Provide the Google API key for authentication
)
```

```py
# Import the AIMessage class currently will be used for typing
from langchain_core.messages.ai import AIMessage

ai_msg: AIMessage = llm.invoke("Hi?")
```

```py
from typing_extensions import TypedDict

class FirstLLMAgentCall(TypedDict):
    prompt: str
    output: str
```

```py
def node_1(state: FirstLLMAgentCall):
    print("---Node 1---", state)
    prompt = state["prompt"]
    ai_msg: AIMessage = llm.invoke(prompt)
    return {"output": ai_msg.content}
```

```py
zeeshan_bhai_greet_message = node_1(FirstLLMAgentCall(prompt="Hello from UMT"))
print(zeeshan_bhai_greet_message)
```

```py
from IPython.display import Image, display # Preview Graph

from langgraph.graph import StateGraph, START, END
from langgraph.graph.state import CompiledStateGraph # type

# Build graph
builder: StateGraph = StateGraph(state_schema=FirstLLMAgentCall)

# Define Nodes
builder.add_node("node_1", node_1)

# Add Edges
builder.add_edge(START, "node_1")
builder.add_edge("node_1", END)

# Compile Graph
graph: CompiledStateGraph = builder.compile()
```

```py
result = graph.invoke({"prompt" : "Motivate me to learn LangGraph"})
print(result)
```

```py
# just another helpter function
import textwrap
from IPython.display import display, Markdown

def to_markdown(text)-> Markdown:
    text : str = text.replace("•", "  *")
    return Markdown(textwrap.indent(text, "> ", predicate=lambda _: True))
```

```py
print("PROMPT: ", result['prompt'])
to_markdown(result['output'])
```

# For Conditional Edge Logic

```py
import random
from typing import Literal

def decide_mood(state: State) -> Literal["node_2", "node_3"]:
    
    # Often, we will use state to decide on the next node to visit
    user_input = state['graph_state'] 
    
    # Here, let's just do a 50 / 50 split between nodes 2, 3
    if random.random() < 0.5:

        # 50% of the time, we return Node 2
        return "node_2"
    
    # 50% of the time, we return Node 3
    return "node_3"
```