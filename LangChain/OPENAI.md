# LangChain Chat models with OpeNAI

- Let's check that your `OPENAI_API_KEY` is set and, if not, you will be asked to enter it.

```py
%%capture --no-stderr
%pip install --quiet -U langchain_openai langchain_core langchain_community tavily-python
```

```py
import os, getpass

def _set_env(var: str):
    if not os.environ.get(var):
        os.environ[var] = getpass.getpass(f"{var}: ")

_set_env("OPENAI_API_KEY")
```

- There are a few standard parameters that we can set with chat models. Two of the most common are:

    - Model: the name of the model
    - Temperature: the sampling temperature

```py
from langchain_openai import ChatOpenAI
gpt4o_chat = ChatOpenAI(model="gpt-4o", temperature=0)
gpt35_chat = ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0)
```

- Chat models in LangChain have a number of default methods. For the most part, we'll be using:

    - Stream: stream back chunks of the response
    - Invoke: call the chain on an input

```py
from langchain_core.messages import HumanMessage

# Create a message
msg = HumanMessage(content="Hello world", name="Lance")

# Message list
messages = [msg]

# Invoke the model with a list of messages 
gpt4o_chat.invoke(messages)
```

- We get an `AIMessage` response above. Also, note that we can just invoke a chat model with a string. When a string is passed in as input, it is converted to a `HumanMessage` and then passed to the underlying model.

```py
gpt4o_chat.invoke("hello world")
```

## Search Tools
Tavily is a search engine optimized for LLMs and RAG, aimed at efficient, quick, and persistent search results. As mentioned, it's easy to sign up and offers a generous free tier. But ofcourse other search tools can be used if you want to modify the code for yourself.

```py
_set_env("TAVILY_API_KEY")
```

```py
from langchain_community.tools.tavily_search import TavilySearchResults
tavily_search = TavilySearchResults(max_results=3)
search_docs = tavily_search.invoke("What is LangGraph?")
```

```py
search_docs
```