# LangChain

LangChain is an Open Source framework that allows developers to combine LLMs
with external data. External data could be a course syllabus or business documentation. 

The basic steps for using LangChain are: 

1. Load our external data (text files, pdf, html, etc)
2. Break the Data into chunks
3. Create Embeddings
4. Add the Embeddings to a Vector Store
5. Augument a model with the new Vectors

Using Langchain: 

```python
from langchain.document_loaders import WebBaseLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.memory import ConversationBufferMemory
from langchain.llms import OpenAI
from langchain.chains import ConversationalRetrievalChain
from langchain.chat_models import ChatOpenAI

# Load external data
url = "https://365datascience.com/upcoming-courses"
loader = WebBaseLoader(url)
raw_documents = loader.load()

# Break Data into Chunks
text_splitter = RecursiveCharacterTextSplitter()
documents = text_splitter.split_documents(raw_documents)

# Create Embeddings
embeddings = OpenAIEmbeddings(openai_api_key = api_key)

# Add Embeddings to Vector Store
vectorstore = FAISS.from_documents(documents, embeddings)

memory = ConversationBufferMemory(memory_key = "chat_history", return_messages=True)

qa = ConversationalRetrievalChain.from_llm(ChatOpenAI(openai_api_key=api_key, 
                                                  model="gpt-3.5-turbo", 
                                                  temperature=0), 
                                           vectorstore.as_retriever(), 
                                           memory=memory)

query = "What is the next course to be uploaded on the 365DataScience platform?"
result = qa({"question": query})
result["answer"]
```
