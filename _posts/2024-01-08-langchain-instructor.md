---
title: PDF Embedding with Instructor & Langchain
date: 2024-01-08 07:30:00 +0700
categories: [Artificial Intelligence, Langchain]
tags: [AI, Langchain, OpenAI, Python, HuggingFace]
---

<br>

In this [previous post](https://helenaferdy.github.io/posts/langchain/), we created a custom GPT that reads through a embedded PDF as an additional context to help better answer the question asked by user, but it depends on OpenAI for embedding as long as to actually make an answer to user using it's GPT 3.5 Turbo LLM. <br>

This time, we'll create something similar but without OpenAI, utlizing Open Source models that's available for free.

<br>

First lets install some dependencies

```python
!pip -q install langchain openai InstructorEmbedding faiss-cpu
```

> 1. Langchain : Framework for text extraction, embedding, vectorstore creation, and many more integration stuff with large language models (LLMs) like OpenAI.
> 2. OpenAI : Official Python library for interacting with OpenAI's API
> 3. InstructorEmbedding : A library to embed text data into vector representations for accurate similarity comparisons.
> 4. FAISS : Library used by Langchain to create and query vectorstores for fast retrieval of relevant text passages.

<br>

Next we import these dependecies as usual

```python
from langchain.vectorstores import Chroma
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.llms import OpenAI, HuggingFaceHub
from langchain.chains.question_answering import load_qa_chain
from langchain.chains import RetrievalQA
from langchain.document_loaders import PyPDFLoader
from langchain.document_loaders import DirectoryLoader
import os
# InstructorEmbedding 
from InstructorEmbedding import INSTRUCTOR
from langchain.embeddings import HuggingFaceInstructEmbeddings
```

<br>

Here we import the HuggingFace API Token as we need it to access the Embedding Model and LLM.
We also throw an OpenAI API Key just for comparing the results after.

```python
os.environ["OPENAI_API_KEY"] = "OPENAI_API_KEY"
os.environ["HUGGINGFACEHUB_API_TOKEN"] = "HUGGINGFACEHUB_API_TOKEN"
```

<br>

Next open the PDF Files, here we connect it to the Google Drive Folder named Others

![x](/static/2024-01-07-langchain/13.png)

```python
# connect to Google Drive
from google.colab import drive
drive.mount('/content/gdrive', force_remount=True)
root_dir = "/content/gdrive/My Drive"
```

```python
loader = DirectoryLoader(f'{root_dir}/Others/', glob="./*.pdf", loader_cls=PyPDFLoader)
documents = loader.load()
```

<br>

Now we split the text into some text chunks with the size of roughfly 512 chars.

```python
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=512, 
    chunk_overlap=20)

texts = text_splitter.split_documents(documents)
```

<br>

And for the Embedding, here we use Instructor-XL ran locally using GPU Cuda

```python
from langchain.vectorstores import FAISS
from langchain.embeddings import HuggingFaceInstructEmbeddings

instructor_embeddings = HuggingFaceInstructEmbeddings(model_name="hkunlp/instructor-xl", 
                                                      model_kwargs={"device": "cuda"})
```

<br>

After that we store the Vector Represented Data inside the FAISS Vector Database named db_instructEmbedd

```python
db_instructEmbedd = FAISS.from_documents(texts, instructor_embeddings)
```

<br>

This is where the locally run Instructor kicks in, for a 327 pages PDF it took around 1 minute and 45 seconds. Not bad

![x](/static/2024-01-07-langchain/13.png)

<br>

Here we set up the retriever to retrieve 3 most relevant chunk data to be processed next

```python
retriever = db_instructEmbedd.as_retriever(search_kwargs={"k": 3})
```

<br>

And now let's test the database by asking a question related to the PDF

```python
## TESTING
docs = retriever.get_relevant_documents("How to login to calabrio one?")
```

<br>

We get the texts used to be the possible context for the LLM

![x](/static/2024-01-07-langchain/11.png)

<br>

Next we choose the LLM, this time we use the FLAN-T5-XXL from google.

```python
llm = HuggingFaceHub(repo_id="google/flan-t5-xxl")
chain = load_qa_chain(llm, chain_type="stuff")
```

<br>

Lets ask some questions

```python
query = "what is the document about?"

docs = db_instructEmbedd.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

```text
CalabrioONE
```

<br>


```python
query = "how to connect calabrio to cisco cucm?"

docs = db_instructEmbedd.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

```text
1.Under Local Web Service Settings, select the Enable Local Web Service checkbox to enable 
API integration on this data server. verify the identity of any one who wants to connect to 
Calabrio ONE. CalabrioONE supports three methods of authentication: Default Calabrio ONE, 
SAML Authentication, and Active Directory Authentication.
```

<br>

```python
query = "tell me how can i reset my calabrio password?"

docs = db_instructEmbedd.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

```text
1.Click Forgot Password? ontheLoginpage. 2.Enter youremailaddress, click Email Password Reset, 
and follow the prompts. An email with instructions on how to reset yourpassword will be sentto youremail account.
```

<br>

```python
query = "explain the licensing model?"

docs = db_instructEmbedd.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

```text
Subscription billing and burstlicensing Perpetual licensing
```

<br>

As we can see, the results are not great. The LLM is not able to utilize the context provided by langchain very well. <br>
Let's try changing the LLM to OpenAI and ask the same questions

<br>

```python
llm = OpenAI()
chain = load_qa_chain(llm, chain_type="stuff")
```

<br>

```python
query = "what is the document about?"

docs = db_instructEmbedd.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

```text
The document is about configuring and managing various aspects of the CalabrioONE application, 
including training, contacts, bulk contact export, and Five9 reports. It also includes information 
about configuring CDR reconciliation for the UCCE ACD.
```

<br>

```python
query = "how to connect calabrio to cisco cucm?"

docs = db_instructEmbedd.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

```text
 To connect Calabrio to Cisco CUCM, you will need to enable Web Access over HTTP on the devices you plan to use 
 for the Cisco IP Phone Service Recording Controls. You will also need to have a Calabrio ONE Data Server. 
 Once you have these prerequisites, you can configure the IP Phone Service Recording Controls in Calabrio ONE's 
 System Configuration. You will also need to configure the ACD Filtering and Cisco Customer Journey Platform API 
 fields under the Connection Settings for Cisco Webex Contact Center section. After configuring the data server, 
 you will then need to install and configure the Base Data Server to become a Shared Data Server. Finally, 
 you will need to add a new configuration to the data server and enter the necessary information, including the server name.
```

<br>


```python
query = "tell me how can i reset my calabrio password?"

docs = db_instructEmbedd.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

```text
To reset your Calabrio password, you can click on "ForgotPassword?" on the login page and enter your email address. 
Then, click "Email Password Reset" and follow the prompts in the email that will be sent to your email account. 
You can also change your password at any time by clicking on the drop-down arrow next to your username and choosing 
"Change Password." Make sure to follow the password rules, such as not using whitespaces and having a minimum length 
of eight characters with characters from at least three of the four categories listed. If you forget your password 
or your account is locked, you will need to contact an administrator to unlock your account or reset your password.
```

<br>


```python
query = "explain the licensing model?"

docs = db_instructEmbedd.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

```text
Calabrio has two licensing models: subscription billing and burst licensing, and perpetual licensing. 
Subscription billing and burst licensing are for CalabrioONE-hosted cloud deployments, while perpetual
licensing is for on-premise deployments. Licenses are linked to the roles and permissions that users
have in the system, and the access they have to CalabrioONE functionality is based on their role and
associated permissions. Each active user is associated with one or more licenses and can have one or
more roles. These licenses, or "seats," are counted by the number of active users with that licensed
role. Customers must have a set number of seats. Inactive users cannot be licensed or have access to 
CalabrioONE, and when a user is deactivated, their license becomes available to be reassigned to another user. 
There are also data licenses for users with capture functions, such as contact center agents.
```

<br>

It's quite apparent that OpenAI has much better quality at utilizing the context provided by langchain and use it to answer the questions.
So as for now, i think OpenAI's LLM is still the way to go for creating our own custom AI.









