# Techniques that used in LangChain

1. [Streaming](/LangChain/TECHNIQUES.md#1-streaming)
2. [Function/tool calling](/LangChain/TECHNIQUES.md#2-function--tool-calling)
3. [Structured output](/LangChain/TECHNIQUES.md#3-structured-output)
4. [Few-shot prompting](/LangChain/TECHNIQUES.md#4-few-shot-prompting)
5. [Retrieval](/LangChain/TECHNIQUES.md#5-retrieval-strategies-overview)
6. [Text splitting](/LangChain/TECHNIQUES.md#6-text-splitting)
7. [Evaluation](/LangChain/TECHNIQUES.md#7-evaluation)
8. [Tracing](/LangChain/TECHNIQUES.md#8-tracing)

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

## 2. Function / Tool Calling

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

- LLMs can generate structured outputs like JSON, useful for fitting specific schemas.

***`.with_structured_output()`***

- Uses a schema to format output directly.
- **Example:** Generates structured response using Pydantic objects.

```py
class Joke(BaseModel):
    """Joke to tell user."""

    setup: str = Field(description="The setup of the joke")
    punchline: str = Field(description="The punchline to the joke")
    rating: Optional[int] = Field(description="How funny the joke is, from 1 to 10")

structured_llm = llm.with_structured_output(Joke)
structured_llm.invoke("Tell me a joke about cats")
```

***Raw Prompting***

- Use descriptive prompts to guide output format.
- Flexible, but output can vary.

***JSON Mode***

- Supported by models like OpenAI and Mistral.
- Enforces JSON structure via config.

```py
model = ChatOpenAI(
    model="gpt-4o",
    model_kwargs={ "response_format": { "type": "json_object" } }
    )
prompt = ChatPromptTemplate.from_template(
    "Answer the user's question to the best of your ability."
    'You must always output a JSON object with an "answer" key and a "followup_question" key.'
    "{question}"
)

chain = prompt | model | SimpleJsonOutputParser()
chain.invoke({ "question": "What is the powerhouse of the cell?" })
```

***Tool Calling***

- Binds schema using `.bind_tools()` and returns structured arguments.
- Most reliable for complex structured output.

```py
class ResponseFormatter(BaseModel):
    """Always use this tool to structure your response to the user."""

    answer: str = Field(description="The answer to the user's question")
    followup_question: str = Field(description="A followup question the user could ask")

model = ChatOpenAI(
    model="gpt-4o",
    temperature=0,
)

model_with_tools = model.bind_tools([ResponseFormatter])
ai_msg = model_with_tools.invoke("What is the powerhouse of the cell?")
ai_msg.tool_calls[0]["args"]
```

## 4. Few-Shot Prompting

- Few-shot prompting improves model performance by providing example inputs and outputs.

***Generating Examples:***

- **Manual:** Created by people based on task needs.
- **Better Model:** Use outputs from higher-performing models.
- **User/LLM Feedback:** Generate examples from positive interactions or LLM self-evaluation.

```py
examples = [
    {"input":"What is the capital of France?", "output": "The capital of France is Paris."},
    {"input":"Who wrote 'To Kill a Mockingbird'?", "output": "Harper Lee wrote 'To Kill a Mockingbird'?"},
    {"input":"What is the Lasgest planet in our solar system?", "output": "Jupiter is the largest planet in our solar system."},
    {"input":"How does photosynthesis work?", "output": "Photosynthesis is the process by which plants use sunlight, water, and carbon dioxide to create oxygen and energy in the form of sugar."},
    {"input":"What is the boiling point of water?", "output": "The boiling point of water is 100 degree celcius."},
]
```

***Number of Examples:***

- More examples = better performance, but increased costs and latency.
- Balance number based on model, task, and constraints.

```py
documents = [
    Document(page_content=example['input'], metadata={"input": example["input"], "output": example["output"]}) 
    for example in examples

vector_store = FAISS.from_documents(documents, embeddings)

selector = SemanticSimilarityExampleSelector(
    vectorstore=vector_store,
    k=2, # Number of examples to select
)
]
```

***Selecting Examples:***

- Choose examples by random selection, semantic similarity, or token size.
- Best performance often comes from semantic similarity.

***Formatting Examples:***

- Format for single-turn (simple input-output) or multi-turn (correct errors) contexts.
- For chat models, insert as system strings or individual messages.
- Use role labels like "example_user" and "example_assistant" to distinguish actors.

```py
# Define the example prompt template
example_prompt = PromptTemplate(
    input_variables=["input", "output"],
    template="Input: {input}\nOutput: {output}",
)

# Create a Few-Shot Prompt Template using the selector
few_shot_prompt = FewShotPromptTemplate(
    example_selector=selector,
    example_prompt=example_prompt,
    prefix="You are a helpful assistant that provides accurate answers to questions.",
    suffix="Input: {adjective}\nOutput:",
    input_variables=["adjective"],
    example_separator="\n\n",
)
```

***Tool Call Formatting:***

- Different models have unique constraints for tool call examples (ToolMessages or AI Messages).
- Adapt based on model-specific requirements.

```py
chain = LLMChain(
    llm=llm,
    prompt=few_shot_prompt,
    verbose=True 
)
```

## 5. Retrieval Strategies Overview

***1. Query Translation Strategies***

| **Name**        | **When to Use**                                                | **Description**                                                                                                   |
|-----------------|---------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| **Multi-query** | Cover multiple perspectives of a question.                    | Rewrite the user question from different angles and retrieve documents for each.                                  |
| **Decomposition** | Break down a question into smaller subproblems.              | Decompose the question into sub-questions for sequential or parallel solving.                                     |
| **Step-back**   | Higher-level understanding is needed.                          | Ask a general question to retrieve relevant concepts that inform the main query.                                  |
| **HyDE**        | Challenges in retrieving relevant documents.                   | Convert questions into hypothetical documents to enhance retrieval of real documents.                             |

***2. Routing Strategies***

| **Name**            | **When to Use**                                                   | **Description**                                                                                          |
|---------------------|------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|
| **Logical Routing** | Use rules to route input.                                        | Prompt LLM with logical rules to select the appropriate datastore based on query type.                   |
| **Semantic Routing**| Use semantic similarity for routing.                             | Embed the query and choose prompts or databases based on semantic similarity.                             |

***3. Query Construction Strategies***

| **Name**        | **When to Use**                                                            | **Description**                                                                                                        |
|-----------------|---------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------|
| **Text-to-SQL** | For questions needing relational database data.                          | Transform user input into SQL queries for relational databases.                                                         |
| **Text-to-Cypher** | For questions needing graph database data.                             | Convert user input into Cypher queries for graph databases.                                                            |
| **Self Query**  | For metadata-based document retrieval.                                   | Create a semantic query string and metadata filter to fetch specific documents.                                       |

***4. Indexing Strategies***

| **Name**                   | **Index Type**          | **Uses an LLM** | **When to Use**                                                                                        | **Description**                                                                                                                                   |
|----------------------------|------------------------|----------------|------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| **Vector Store**           | Vector Store           | No             | If just starting or needing a simple setup.                                                        | Create embeddings for semantic similarity search.                                                                                                  |
| **ParentDocument**         | Vector Store + Document Store | No             | For distinct information needing combined retrieval.                                                 | Index multiple chunks for documents; retrieve the whole parent based on similar chunks.                                                          |
| **Multi Vector**           | Vector Store + Document Store | Sometimes during indexing | When relevant information needs indexing beyond the text.                                             | Create multiple vectors (e.g., summaries) for improved indexing.                                                                                  |
| **Time-Weighted Vector Store** | Vector Store           | No             | For time-sensitive documents needing recency in retrieval.                                         | Combines semantic similarity with recency based on document timestamps.                                                                            |

***5. Search Optimization Strategies***

| **Name**          | **When to Use**                                               | **Description**                                                                                           |
|-------------------|--------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| **ColBERT**       | When higher granularity embeddings are needed.               | Uses contextually influenced embeddings for each token in documents and queries to improve similarity scores. |
| **Hybrid Search** | When combining keyword and semantic similarity.               | Merges keyword and semantic searches for optimized results.                                              |
| **Maximal Marginal Relevance (MMR)** | When diversifying search results is needed.  | Diversifies search results by avoiding redundant documents.                                              |

***6. Post-Processing Strategies***

| **Name**                | **Index Type**        | **Uses an LLM** | **When to Use**                                                                 | **Description**                                                                                                                                                        |
|-------------------------|----------------------|----------------|--------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Contextual Compression** | Any                  | Sometimes       | If retrieved documents contain too much irrelevant information.                 | Extracts the most relevant information from retrieved documents using LLMs or embeddings.                                                                          |
| **Ensemble**            | Any                  | No             | If combining multiple retrieval methods.                                        | Fetches documents from various retrievers and merges results.                                                                                                       |
| **Re-ranking**          | Any                  | Yes            | When needing to rank documents based on relevance.                            | Ranks documents from most to least relevant based on the query, useful when combining multiple retrieval methods.                                                    |

***7. Generation Strategies***

| **Name**             | **When to Use**                                           | **Description**                                                                                         |
|----------------------|----------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| **Self-RAG**         | When fixing hallucinations or irrelevant content is needed. | Iteratively checks document relevance and quality to build an accurate response, correcting errors as needed. |
| **Corrective-RAG**   | For fallback when low relevance documents are retrieved. | Implements fallback mechanisms (e.g., web search) when documents aren’t relevant, enhancing retrieval quality. |

## 6. Text splitting

- LangChain provides a variety of text splitters to ***break down large text into smaller, manageable chunks*** for improved processing.

***Text Splitters Overview***

| **Name**                    | **Classes**                                              | **Splits On**                 | **Adds Metadata** | **Description**                                                                                         |
|-----------------------------|---------------------------------------------------------|------------------------------|------------------|---------------------------------------------------------------------------------------------------------|
| **Recursive**               | `RecursiveCharacterTextSplitter`, `RecursiveJsonSplitter `  | List of user-defined characters | ❌                 | Recursively splits text while keeping related parts together. Recommended starting point for splitting.  |
| **HTML**                    | `HTMLHeaderTextSplitter`, `HTMLSectionSplitter`            | HTML-specific characters      | ✅                 | Splits text based on HTML tags, retaining metadata about the source HTML structure.                      |
| **Markdown**                | `MarkdownHeaderTextSplitter`                             | Markdown-specific characters  | ✅                 | Splits text based on Markdown headers and elements, adding metadata for context.                         |
| **Code**                    | Multiple classes for different languages                | Language-specific syntax      | ❌                 | Splits text based on syntax of various programming languages (e.g., Python, JS). Ideal for code analysis.|
| **Token**                   | Multiple classes                                        | Tokens                       | ❌                 | Splits text on tokens using various tokenization methods.                                               |
| **Character**               | `CharacterTextSplitter`                                   | User-defined characters       | ❌                 | Simple method that splits text on a specified character, useful for straightforward splitting needs.     |
| **Semantic Chunker**        | `SemanticChunker`                                          | Sentences                     | ❌                 | Splits text into sentences, then merges similar ones to form coherent chunks. Useful for preserving context. |
| **Integration: AI21 Semantic** | `AI21SemanticTextSplitter`                                 | Semantic topics               | ✅                 | Splits based on distinct topics, forming coherent pieces of text and retaining metadata. Ideal for topic-focused segmentation. |

***Recommendations:***

- Start with **Recursive Splitter** for general splitting.
- Choose **HTML** or **Markdown Splitters** for structured documents.
- Use **Code Splitters** for programming text.
- Opt for **Semantic Splitters** when context preservation is critical.

## 7. Evaluation

***Overview:***
- **Dataset:**    Provides predefined examples as inputs.
- **Application:**    Runs and generates outputs.
- **Evaluator:**    Compares actual outputs with expected results.
- **Score:**    Quantifies performance based on evaluation criteria.

***LangSmith Support:***
- **Dataset Management:**    Simplifies tracing and annotation.
- **Evaluation Framework:**    Helps define metrics and run evaluations.
- **Tracking & Scheduling:**    Enables continuous evaluation via CI/Code integration.

![ss](/LangChain/Evaluation.png)

This structured evaluation process ensures LLM applications meet desired quality standards and perform as expected over time.

## 8. Tracing

***Overview:***
- **Definition:** A trace is a record of all steps your application takes from input to output.

- **Components:** Consists of individual steps called runs, which include:
   - Model calls
   - Retriever interactions
   - Tool invocations
   - Sub-chain executions

- **Purpose:** Provides observability within chains and agents, helping diagnose issues and understand the flow of execution.

***Key Benefits:***
- Pinpoints where problems occur.
- Analyzes step-by-step performance.
- Enhances transparency in complex workflows.