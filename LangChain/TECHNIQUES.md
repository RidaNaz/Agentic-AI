# Techniques that used in LangChain

1. [Streaming](/LangChain/TECHNIQUES.md#1-streaming)
2. [Function/tool calling](/LangChain/TECHNIQUES.md#)
3. [Structured output](/LangChain/TECHNIQUES.md#)
4. [Few-shot prompting](/LangChain/TECHNIQUES.md#)
5. [Retrieval](/LangChain/TECHNIQUES.md#)
6. [Text splitting](/LangChain/TECHNIQUES.md#)
7. [Evaluation](/LangChain/TECHNIQUES.md#)
8. [Tracing](/LangChain/TECHNIQUES.md#)

## 1. Streaming

- LLMs ***generate outputs iteratively***, making it possible to display intermediate results before the complete response is ready.
- This approach **improves the user experience (UX)** by reducing latency when building applications that use long-running LLM calls or multiple reasoning steps.

***Streaming Methods***

These methods provide a simple interface for streaming output.
   - **`.stream()`:**   Synchronous streaming method.
   - **`.astream()`:**   Asynchronous streaming variant.

```py
from langchain_anthropic import ChatAnthropic

model = ChatAnthropic(model="claude-3-sonnet-20240229")
for chunk in model.stream("what color is the sky?"):
    print(chunk.content, end="|", flush=True)
```

- Each output chunk type varies based on the component, such as `AIMessageChunks` for chat models.
- For components without native streaming support, `.stream()` can still be used, but only a single chunk will be yielded.

***`.astream_events()` Method***

- This method is ideal for capturing both intermediate and final results, making it useful for complex chains.
- Returns an iterator over different types of events, allowing for more granular control.

```py
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_anthropic import ChatAnthropic

model = ChatAnthropic(model="claude-3-sonnet-20240229")
prompt = ChatPromptTemplate.from_template("tell me a joke about {topic}")
parser = StrOutputParser()
chain = prompt | model | parser

async for event in chain.astream_events({"topic": "parrot"}, version="v2"):
    kind = event["event"]
    if kind == "on_chat_model_stream":
        print(event, end="|", flush=True)
```

***Callbacks for Streaming***

- Callbacks are the lowest-level method for handling streaming in LangChain.
- Used to manage token-level events like `on_llm_new_token` and `on_llm_end`.
- While flexible, they require more manual management and can be cumbersome.

***Understanding Tokens***

- **Tokens:** The basic units LLMs use to process text. These can be full words or parts of words.
- Language models use tokens to improve efficiency and better understand context and structure.
- **Example:** "LangChain is cool!" might be split into five separate tokens.

## 2. Function / tool calling

- Tool calling (or function calling) allows chat models to generate structured output matching a schema but doesn’t execute the tool itself—the user controls this execution. Key features include:

***What is Tool Calling?***

- The model generates arguments for a tool, but running it is managed by the user.
- **Use Case:** Extract structured data (e.g., dates) from text without further action.

***Supported LLM Providers***

- Includes OpenAI, Anthropic, Cohere, Google, Mistral, and local models via Ollama.

***LangChain's Standard Interface***

- **ChatModel.bind_tools():** Defines which tools the model can use.
- **AIMessage.tool_calls:** Accesses tool call details in the AIMessage.

***Tool Calling Flow***

- Generate tool calls based on a query.
- Invoke tools using generated arguments.
- Format results as ToolMessages.
- Pass messages back for the final answer or more calls.

![ss](/LangChain/tool_calling_flow.png)

## 3. Structured Output

