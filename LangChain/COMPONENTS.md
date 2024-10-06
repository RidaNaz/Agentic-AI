# Components

1. [Chat Models](/LangChain/COMPONENTS.md#1-chat-models)
2. [LLM](/LangChain/COMPONENTS.md#2-llm)
3. [Messages](/LangChain/COMPONENTS.md#3-messages)
4. [Prompt Templates](/LangChain/COMPONENTS.md#4-prompt-templates)
5. [Example Selectors](/LangChain/COMPONENTS.md#5-example-selectors)
6. [Output Parser](/LangChain/COMPONENTS.md#6-output-parser)
7. [Chat History](/LangChain/COMPONENTS.md#7-chat-history)
8. [Documents](/LangChain/COMPONENTS.md#8-documents)
9. [Document Loaders](/LangChain/COMPONENTS.md#9-document-loaders)
10. [Text Splitters](/LangChain/COMPONENTS.md#10-text-splitters)
11. [Embedding Models](/LangChain/COMPONENTS.md#11-embedding-models)
12. [Vector Stores](/LangChain/COMPONENTS.md#12-vector-stores)
13. [Retriever](/LangChain/COMPONENTS.md#13-retriever)

## 1. Chat models

- Chat models handle inputs and outputs as sequences of messages, unlike traditional LLMs, which use plain text.
- They support different roles (e.g., *system, AI, and human*) to distinguish message types.

***LangChain Functionality:***

- While LangChain does not host chat models, it supports third-party integrations.
- Chat models can accept a string input, converting it into a `HumanMessage`.

***Parameters for Configuration:***

- Key parameters include `model`, `temperature`, `timeout`, `max_tokens`, `stop sequences`, `max_retries`, `api_key`, and `base_url`. (Not all parameters are supported across all providers)

***Multimodal Capabilities:***

- Some chat models support multimodal inputs like images, audio, and video.
- The implementation is evolving, and current support is limited mostly to image inputs.

## 2. LLM

- Traditional LLMs (Language Models) take a string as input and return a string as output.
- While newer models are more commonly structured as chat models.

***LangChain Functionality:***

- LangChain supports LLMs through third-party integrations.
- It provides flexibility to handle message-based inputs by converting them into strings, allowing LLMs to have a similar interface as chat models.

***Recommendation:***

- For newer, more complex use cases, it’s recommended to use chat models, even if the task is not explicitly chat-related.

## 3. Messages

- Messages have a role, content, and optional `response_metadata`.
- Roles indicate who is sending the message, such as:
    - `user` (HumanMessage)
    - `assistant` (AIMessage)
    - `system` (SystemMessage)
    - `tool` (ToolMessage)

***Content Types:***
Content can be:
   - A ***string*** (common for text input)
   - A ***list of dictionaries*** (used for multimodal inputs like images)

***Message Types:***

- **HumanMessage:**  Represents user input.
- **AIMessage:**  Represents AI responses and can contain `response_metadata` (e.g., token usage, log-probs).
- **SystemMessage:**  Provides instructions to the model (e.g., defining behavior).
- **ToolMessage:**  Contains results from tools, with fields like `tool_call_id` and artifact.
- **FunctionMessage:**  Legacy type, now replaced by ToolMessage.

***Tool Calls:***

- `tool_calls` are part of `AIMessage` and contain information on which tool to call, arguments, and IDs.

## 4. Prompt Templates

- Prompt templates are used to translate user inputs and parameters into instructions for language models, guiding responses to be context-aware and coherent.
- They can generate either a *single string* or a *list of structured messages*.

***Key Concepts:***

- **Input:** Takes a dictionary where keys are variables to fill in the template.
- **Output:** Produces a `PromptValue` that can be cast to a string or a list of messages, making it flexible for both LLMs and chat models.

***Types of Prompt Templates:***

- **String PromptTemplates:** Used to format a single string (e.g., `"Tell me a joke about {topic}"`).
- **ChatPromptTemplates:** Used to format a list of messages (e.g., a combination of system and user messages).

***MessagesPlaceholder:***

- Enables inserting a list of dynamic messages at a specified spot in the template.
- Useful for handling multiple inputs, such as passing a conversation history into a specific position.

***Alternative Method:***

- Instead of `MessagesPlaceholder`, a placeholder can be defined using `{variable_name}` directly in the template.

[Learn how to implement the Prompt Templates](/LangChain/PROMPTS.md#implementing-the-prompt-templates)

## 5. Example Selectors

- Example Selectors are used in ***few-shot prompting*** to enhance model performance by providing specific examples of how the model should respond.
- These examples serve as guidance, improving the relevance and quality of the generated outputs.

***Functionality:***

- ***Static vs. Dynamic:***
    - **Static:**  Hardcoded examples included directly in the prompt.
    - **Dynamic:**  Selectors can dynamically choose and format examples based on context.

***Usage:***

- Example Selectors are classes designed to pick the most appropriate examples and format them into the prompt.

## 6. Output Parser

- Output parsers transform the text output from a model into structured formats for easier downstream processing.
- They are useful when working with structured data or normalizing output for better readability and usability.

***Recommendation:***

- With the rise of models supporting function/tool calling, it’s recommended to use those for structured outputs instead of traditional parsers.

***Key Features of Output Parsers:***

- **Name:**   Identifies the parser type.
- **Supports Streaming:**   Indicates if the parser can handle real-time streaming outputs.
- **Format Instructions:**   Specifies if the parser includes format guidelines.
- **Calls LLM:**   Some parsers can interact with LLMs to correct misformatted outputs.
- **Input/Output Types:**   Defines the expected input (string/messages) and resulting output format.

***Usage:***

- Choose an appropriate parser based on your needs, such as handling schema-specific data, managing streaming - outputs, or ensuring compatibility with specific LLM configurations.

## 7. Chat History

- Chat History is used to maintain a record of inputs and outputs in conversational interfaces, ensuring the context of previous interactions is retained.
- It enables the model to refer back to earlier messages, improving coherence and relevance in ongoing conversations.

***Functionality:***

- Implemented as a **ChatHistory class**, which wraps around a chain.
- Tracks and stores conversation history in a **message database**.
- Subsequent interactions load these stored messages as part of the input, preserving context.

***Use Case:***

- Ideal for applications requiring context retention, such as chatbots or interactive agents.

## 8. Documents

- A Document object in LangChain represents a piece of information with *two main attributes*:
    - `page_content`:   Contains the main content as a string.
    - `metadata`:   A dictionary that holds related metadata, such as document ID, file name, source information, and more.

***Purpose:***

- Useful for *storing, tracking, and processing documents* with associated metadata for tasks like *retrieval, summarization, and analysis*.

## 9. Document Loaders

- Document Loaders are classes designed to load Document objects from various data sources, such as Slack, Notion, Google Drive, and more.

***Functionality:***

- Each loader has its own specific parameters tailored to the source it integrates with.
- All loaders can be invoked uniformly using the `.load` method.

***Example Use Case:***

An example of loading data using a CSV loader:

```py
from langchain_community.document_loaders.csv_loader import CSVLoader

loader = CSVLoader(
    ...  # Integration-specific parameters here
)
data = loader.load()
```

## 10. Text Splitters

- Text Splitters are used to transform loaded documents into ***smaller, manageable chunks*** that fit within a model’s context window, especially when dealing with long texts.

***Functionality:***

- They facilitate splitting, combining, filtering, and manipulating documents for better application compatibility.
- The goal is to maintain semantically related text together while managing the size of chunks.

***How It Works:***

- **Splitting:** The text is divided into small, meaningful chunks (often at the sentence level).
- **Combining:** Smaller chunks are combined until a specified size is reached, creating a new chunk. Overlap may be introduced to preserve context between chunks.

***Customization:***

- Text splitters can be customized based on:
   - **Splitting Criteria:** Defining how the text is split.
   - **Chunk Size Measurement:** Setting the criteria for measuring chunk sizes.

## 11. Embedding Models

- Embedding Models generate ***vector representations of text***, allowing for *mathematical operations* that capture the *semantic meaning of the text*. This enables capabilities such as *natural language search* and *context retrieval.*

***Functionality:***

- By converting text into vectors (arrays of numbers), embedding models help identify **semantically similar pieces of text**, which enhances the performance of language models (LLMs) in *responding to queries*.

***Embeddings Class:***

- The Embeddings class in LangChain serves as a standard interface for various embedding model providers *(e.g., OpenAI, Cohere, Hugging Face)* and local models.

***Included Methods:***

- These methods are separate due to different requirements for embedding documents versus queries.
   - **Embedding Documents:** For embedding multiple texts.
   - **Embedding Query:** For embedding a single text.

## 12. Vector stores

- Vector stores are designed to ***store and search unstructured data by embedding it into vectors***, which allows for efficient similarity searches during query time.

***Functionality:***

They enable:
   - **Embedding Storage:** Keeping the resulting embedding vectors.
   - **Vector Search:** Retrieving the most similar embedding vectors to a given query by first embedding the query itself.

***Metadata Support:***

- Most vector stores can also hold metadata related to embedded vectors, which allows users to filter the results based on this metadata before conducting similarity searches.

***Integration:***

- Vector stores can be easily integrated into the retriever interface:

```py
vectorstore = MyVectorStore()
retriever = vectorstore.as_retriever()
```

## 13. Retriever

- A retriever is an interface designed to ***return documents in response to an unstructured query***.

***Functionality:***

- Unlike vector stores, retrievers:
   - **Do not need to store documents:** They focus solely on retrieving relevant documents based on the provided query.
   - Can be utilized for various types of document retrieval, such as:
      - Vector stores
      - Wikipedia searches
      - Amazon Kendra

***Input and Output:***

- Retrievers accept a string query as input and return a list of Document objects as output.

***Flexibility:***

- The retriever interface is versatile, making it applicable across different retrieval methods and data sources.

[Retrieval Strategies Overview](/LangChain/TECHNIQUES.md#5-retrieval-strategies-overview)