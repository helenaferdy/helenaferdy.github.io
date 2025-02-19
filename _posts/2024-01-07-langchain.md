---
title: PDF Embedding with Langchain & OpenAI
date: 2024-01-07 07:30:00 +0700
categories: [Other, Artificial Intelligence]
tags: []
---

<br>

PDF Embedding is a process to extract data from PDFs and embed it into a vector represented data, which then is used as context for an LLM to answer the question the user asked.

<br>
<br>

First of all, we need to install some dependencies


```python
!pip install langchain
!pip install openai
!pip install PyPDF2
!pip install faiss-cpu
```

> 1. Langchain : Framework for text extraction, embedding, vectorstore creation, and many more integration stuff with large language models (LLMs) like OpenAI.
> 2. OpenAI : Official Python library for interacting with OpenAI's API
> 3. PyPDF2 : Python library for reading and manipulating PDF files
> 4. FAISS : Library used by Langchain to create and query vectorstores for fast retrieval of relevant text passages.

<br>

Then import it to the python code

```python
from PyPDF2 import PdfReader
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.text_splitter import CharacterTextSplitter
from langchain.vectorstores import FAISS
import os
```

<br>

In this project, we'll use OpenAI's embedding and LLM models so we need to have an API key

```python
os.environ["OPENAI_API_KEY"] = "THE_OPENAI_KEY"
```

<br>

Next declare the PDF used that will be processed. Here we'll use a 11 Pages document titled "Setting Up Your Firewall" by Palo Alto

![x](/static/2024-01-07-langchain/00.png)

```python
pdfreader = PdfReader('palo.pdf')
```

<br>

Here's where the PDF being extracted to become a single line of raw texts

```python
raw_text = ''
for i, page in enumerate(pdfreader.pages):
    content = page.extract_text()
    if content:
        raw_text += content
```

If we print the raw_text variable, we can see the content of the PDF document

![x](/static/2024-01-07-langchain/01.png)

<br>

Next we split this single line of text into a couple of chunk texts

```python
text_splitter = CharacterTextSplitter(
    separator = "\n",
    chunk_size = 500,
    chunk_overlap  = 200,
    length_function = len,
)
texts = text_splitter.split_text(raw_text)
```

And the content has been stored as an array of chunk text

![x](/static/2024-01-07-langchain/01a.png)

<br>

After that, we'll embed the chunk texts to become vector representations. These embeddings capture semantic meaning and relationships between words and phrases, enabling similarity-based search and machine understanding of text.

```python
embeddings = OpenAIEmbeddings()
```

<br>

Next we create a FAISS Vectorstore, a specialized database optimized for storing and searching high-dimensional vectors. <br>
This line establishes a searchable database of text embeddings, enabling efficient retrieval of relevant information based on semantic similarity

```python
document_search = FAISS.from_texts(texts, embeddings)
```

<br>

Now that we finish dealing with our PDF, lets import load_qa_chain to handle question-answering tasks using large language models (LLMs) which in this case is OpenAI's.

```python
from langchain.chains.question_answering import load_qa_chain
from langchain.llms import OpenAI
```

<br>

Finally, let's ceate a question-answering chain using OpenAI's LLMs, and procedd to query our question. <br>
The chain will use document_search FAISS object to find text passages semantically similar to the query.

```python
chain = load_qa_chain(OpenAI(), chain_type="stuff")

query = "i heve PAN-OS 6.0. how to i get to 7.0.x"
docs = document_search.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

<br>

And this is the answer of the query

```text
Based on the context provided, it appears that to upgrade from PAN-OS 6.0 to 7.0.x, 
you would need to first install the next major version, 6.1, before being able to 
upgrade to 7.0 and subsequent versions. You would also need to make sure that the 
desired release is reached and that the installation package is available and has 
not been updated with a newer version.
```

Whic is taken straight from the document

![x](/static/2024-01-07-langchain/02.png)

<br>

Let's try to query other question, this time its about this part of document which is about licensing

![x](/static/2024-01-07-langchain/03.png)

<br>

```python
query = "explain about the licensing system"
docs = document_search.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

<br>

And the LLM is able to answer the query pretty well.

```text
The licensing system controls the different functions of the system and allows for 
the download of content, application updates, and software upgrades. Different licenses 
are required for different functions, such as the support license for software and 
AppID updates, the Threat Prevention license for virus, threat, and malware signatures, 
and the URL license for URL categories used in security policies. The device must be 
registered on the support portal and have the necessary licenses added in order for 
updates to be downloaded. Once the licenses have been successfully added, the system 
can retrieve a list of available updates and display them for download. If the device 
has not been registered, an authorization code can be used to activate a license through 
a Palo Alto sales contact. Once the licenses are added, updates can be downloaded through 
the Dynamic Updates section of the Device tab.
```

<br>

Now if we enable verbosity with command "verbose=True", we can sorta see how langchain is able to provide PDF as context for the LLM to answer our questions

```python
chain = load_qa_chain(OpenAI(), chain_type="stuff", verbose=True)

query = "explain about the licensing system"
docs = document_search.similarity_search(query)
chain.run(input_documents=docs, question=query)
```

<br>

As we can see below, langchain provides a chunk of the PDF's content that deemed to be relevant to the query asked so the LLM can use it as an added context to be able to anwer the questions.

```text
> Entering new StuffDocumentsChain chain...


> Entering new LLMChain chain...
Prompt after formatting:
Use the following pieces of context to answer the question at the end. 
If you don't know the answer, just say that you don't know, don't try to make up an answer.

Tobeallowedtodownloadcontentandapplicationupdatesorsoftwareupgrades,the
systemneedstobelicensed.Variouslicensescontrolthedifferentfunctionsofthesystem,
sothe—
SupportlicenseentitlesthesystemtosoftwareandAppIDupdates
ThreatPreventionlicenseaddsvirus,threatsandmalwaresignatures
URLlicenseenablesURLcategoriesforuseinsecuritypolicies
Ifthedevicehasnotbeenregisteredonthesupportportalyet,pleasefollowthesestepsto
registerthedevice:HowtoRegisteraPaloAltoNetworksDevice,Spare,Traps,orVM-
SeriesAuth-Code

Afterthelicenseshavebeensuccesfullyadded,theLicensespagelookssimilartothis:
Nowyou'rereadytostartupdatingthecontentonthisdevice,sonavigatetotheDevicetab,
thenselectDynamicUpdatesfromtheleftpane.
Thefirsttimethispageopens,therewillbenovisiblepackagesfordownload.Thesystem
willfirstneedtofetchalistofavailableupdatesbeforeitcandisplaywhichonesare
available,soselectCheckNow.
2016-09-12_12-51-00.jpg
Whenthesystemretrievesalistofavailableupdates,theApplicationsandThreatspackage

added,goaheadandselectRetrieveLicensekeysfromlicenseserver.
Ifthedevicewasregisteredbutnolicensesaddedyet,selectActivatefeatureusing
authorizationcodetoactivatealicensethroughitsauthorizationcode,whichyouwillhave
receivedfromyourPaloAltosalescontact.
2016-09-12_11-48-47.jpg
Afterthelicenseshavebeensuccesfullyadded,theLicensespagelookssimilartothis:
Nowyou'rereadytostartupdatingthecontentonthisdevice,sonavigatetotheDevicetab,
thenselectDynamicUpdatesfromtheleftpane.

aheadandconnectthecables:
ConnectInterface1totherouter
ConnectInterface2totheswitch
ConnecttheManagment(mgmt)interfacetotheswitch
YoushouldbeabletoconnecttothemanagementIPfromthenetwork,andyoushouldbe
abletosurfouttotheInternet.
2015-10-14_13-17-29.png
2.Preparingthelicensesandupdatingthesystem
Tobeallowedtodownloadcontentandapplicationupdatesorsoftwareupgrades,the
systemneedstobelicensed.Variouslicensescontrolthedifferentfunctionsofthesystem,
sothe—

Question: explain about the licensing system
Helpful Answer:

> Finished chain.

> Finished chain.
 The licensing system for this device requires various licenses to control different 
 functions of the system. These licenses include the Support license, which allows for 
 software and application updates, the Threat Prevention license, which adds virus, 
 threat, and malware signatures, and the URL license, which enables URL categories 
 for use in security policies. To download content and updates, the device must be 
 registered on the support portal and the appropriate licenses must be added. 
 Once the licenses are successfully added, the device can be updated by navigating 
 to the Device tab and selecting Dynamic Updates. If the device has not been registered, 
 it can be registered by following the steps outlined in the "How to Register a Palo Alto 
 Networks Device, Spare, Traps, or VM-Series Auth-Code" guide. Once the licenses are added, 
 the licenses page will show a list of available updates, which can be retrieved from 
 the license server. If the device was registered but no licenses were added, an 
 authorization code can be used to activate a license. Once the licenses are successfully 
 added, the device is ready to start updating its content.
```

<br>

Interesting stuff....