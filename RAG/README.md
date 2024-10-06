# RAG (Retrieval Augmented Generation)

- RAG is a technique for augmenting LLM knowledge with additional data.
- A typical RAG application has two main components:

## 1. Indexing

*  ### Load

- We need to first load the blog post contents.
- We can use [DocumentLoaders](/LangChain/COMPONENTS.md#9-document-loaders) for this, which are objects that load in data from a source and return a list of [Documents](/LangChain/COMPONENTS.md#8-documents).

* ### Split

- To handle too long context we’ll split the Document into chunks for embedding and vector storage.
- **[TextSplitter](/LangChain/TECHNIQUES.md#6-text-splitting):** Object that splits a list of Documents into smaller chunks. Subclass of DocumentTransformers.
- **DocumentTransformer:** Object that performs a transformation on a list of Document objects.

* ### Store

- **Embed Split Document:** Convert each text chunk into an embedding vector.
- **Vector Store:** Insert embeddings into a vector database for similarity search.

## 2. Retrieval and Generation

* ### Retrieve

- LangChain defines a Retriever interface which wraps an index that can return relevant Documents given a string query.
-  Any VectorStore can easily be turned into a Retriever with `VectorStore.as_retriever()`

* ### Generate

- Put it all together into a chain that takes a question, retrieves relevant documents, constructs a prompt, passes that to a model, and parses the output.

***Components Used:***

- **Retriever:**   Retrieves relevant documents from a knowledge base.
- **Formatter:**   Converts retrieved documents into a formatted context.
- **Prompt:**   Uses predefined or custom templates to structure the query.
- **LLM:**   Generates an answer based on the context.
- **Output Parser:**   Extracts the string content from the LLM’s response.

