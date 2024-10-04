# Langchain

LangChain is a framework for building applications with large language models (LLMs) using modular components.

## Components

### Models
* LLMs
* Chat Models
* Text Embedding Models

### Prompts
* [Prompts Templates](/LangChain/PROMPTS.md)
* Output Parser
    - Retry / fixing logic
* Example Selectors

### Indexes
* Document Loaders
* Text Splitters
* Vector Stores
* Retrievers

### Chains
* Prompt + LLM + Output Parsing
* Can be used as building  blocks for longer chains
* More application specific chains


## 1. Memory

* ### ConversationBufferMemory:
Stores the entire conversation history in a buffer, useful for applications needing full context.

* ### ConversationBufferWindowMemory:
Keeps only a recent window of interactions (e.g., last 5 messages), allowing for focused context while reducing memory usage.

* ### ConversationTokenBufferMemory:
Similar to buffer memory but limits history based on the number of tokens, optimizing memory for token-based models.

* ### ConversationSummaryMemory:
Summarizes past interactions into a concise format, retaining key points instead of full transcripts, ideal for long conversations.

## 2. Chains
Chains are sequences of actions or components used to process inputs and produce structured outputs. They link different modules like language models, prompts, memory, and logic together, enabling complex workflows.

### Types of Chains:
- **Simple Chains:**    Single input → Single output (e.g., Question Answering).
- **Sequential Chains:**    Multi-step operations where each output serves as the next input.
- **Router Chains:**    Routes input to different chains based on user intent.
- **Transform Chains:**    Modify outputs in-between steps.

## 3.  Evaluation
Evaluation refers to the process of assessing the performance and accuracy of language model responses. It helps determine how well the chains, models, and workflows perform in real-world scenarios.

### Types of Evaluation:
- **Human Evaluation:**     Direct feedback from users.
- **Automated Metrics:**     Using predefined benchmarks like BLEU, ROUGE, or F1-scores.
- **Comparative Evaluation:**     Comparing outputs from multiple models or configurations.

## 4. Messages
These are message types used to structure conversations in a clear and contextually organized manner.

- **HumanMessage:**   Represents input from the user (e.g., a question or command).
- **AIMessage:**   Response generated by the AI model, such as ChatGPT’s replies.
- **SystemMessage:**   Provides instructions or sets context for the AI model (e.g., "You are a helpful assistant").

## 5. Example Selectors
- **LengthBasedExampleSelector:**   Selects examples based on their length, suitable for contexts needing either brief or detailed responses.

- **SemanticSimilarityExampleSelector:**   Chooses examples that are semantically similar to a given input, ensuring relevance to user queries.

- **MaximizingRelevanceExampleSelector:**   Focuses on selecting the most relevant examples for a task, prioritizing contextual appropriateness.

- **RandomExampleSelector:**   Randomly selects examples from a set, providing diverse responses to enhance user engagement.


# Relevant LangChain Classes for Gemini Models:

## 1. GoogleGenerativeAIEmbeddings:

- This class is used to create embeddings from Gemini models like `textembedding-gecko@001` for semantic search and other embedding-based applications.
- Usage: It allows embedding of documents, text, and other content using the Google Gemini API.

## 2. ChatGoogleGenerativeAI:

- Used to integrate conversational capabilities of Gemini models, such as `gemini-1.5-flash`.
- Supports *model configuration*, *conversation history*, and *response customization*.














