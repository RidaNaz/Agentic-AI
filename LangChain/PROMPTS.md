## Key Concepts to Understand:

#### Prompt Templates:
- Used to format user input into a structured format that an LLM can work with.

#### ChatPromptTemplate:
- A class that helps create structured prompts, which can include both system and user messages.

#### MessagesPlaceholder:
- Allows us to dynamically insert messages into the prompt template.

#### RunnableWithMessageHistory:
- A utility that retains and manages chat history for maintaining context between sessions.

# Implementing the Prompt Templates

## 1. Setting Up a Basic ChatPromptTemplate:

- `ChatPromptTemplate` allows the separation of different input components (e.g., system, user messages).
- With placeholders like `MessagesPlaceholder`, you can dynamically change what gets passed to the LLM.

```py
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain.schema import HumanMessage

# Create a simple ChatPromptTemplate
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant. Answer all questions to the best of your ability."),
        MessagesPlaceholder(variable_name="messages"),
    ]
)

# Define the chain by connecting the prompt with a language model (model should be instantiated beforehand)
chain = prompt | model

# Invoke the chain with some input
response = chain.invoke({"messages": [HumanMessage(content="Hi! I'm Bob")]})

print(response.content)
```

## 2. Adding Dynamic Inputs to the Prompt Template:

- You can add variables like `{language}` to customize responses based on user preference.

```py
# Modify the prompt to include a dynamic `language` variable
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant. Answer all questions to the best of your ability in {language}."),
        MessagesPlaceholder(variable_name="messages"),
    ]
)

# Redefine the chain with the updated prompt
chain = prompt | model

# Invoke the chain, specifying the language
response = chain.invoke(
    {"messages": [HumanMessage(content="Hi! I'm Bob")], "language": "Spanish"}
)

print(response.content)  # Output: ¡Hola, Bob! ¿En qué puedo ayudarte hoy?
```

## 3. Managing Conversation History:

- `RunnableWithMessageHistory` ensures the model retains context across multiple interactions, making it ideal for building conversational agents.

```py
from langchain_core.runnables import RunnableWithMessageHistory

# Initialize RunnableWithMessageHistory to track conversation history
with_message_history = RunnableWithMessageHistory(
    chain,
    get_session_history,  # A function that retrieves session history
    input_messages_key="messages",
)

# Define some configurations (e.g., session ID for the chat)
config = {"configurable": {"session_id": "abc11"}}

# Pass in inputs along with config to maintain session
response = with_message_history.invoke(
    {"messages": [HumanMessage(content="Hi! I'm Todd")], "language": "Spanish"},
    config=config,
)

print(response.content)  # Output: ¡Hola Todd! ¿En qué puedo ayudarte hoy?

# Model retains context and responds accurately based on session history
response = with_message_history.invoke(
    {"messages": [HumanMessage(content="What's my name?")], "language": "Spanish"},
    config=config,
)

print(response.content)  # Output: Tu nombre es Todd.
```

