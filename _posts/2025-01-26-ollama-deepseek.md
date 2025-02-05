---
title: Deepseek with Ollama
date: 2025-01-26 07:30:00 +0700
categories: [Other, Artificial Intelligence]
tags: [AI, Ollama, Deepseek]
---

Ollama is a platform that enables us to run large language models (LLMs) locally on our machine. It can be used to deploy DeepSeek, a powerful LLM capable of tasks like text generation, summarization, and conversational AI. To make interacting with DeepSeek more intuitive, OpenWebUI can be used to serve as the front-end, offering a user-friendly web interface for inputting queries and receiving responses. 

<br>

## Ollama

First lets download [Ollama](https://ollama.com), here we'll run it on Ubuntu Server

![x](/static/2025-01-26-ollama-deepseek/01.png)

<br>

Run the command on the server

![x](/static/2025-01-26-ollama-deepseek/02.png)

<br>

## Deepseek

Now that Ollama is installed, we can proceed with getting the LLM to use, which in this case its DeepSeek

![x](/static/2025-01-26-ollama-deepseek/03.png)

<br>

Run the "ollama run deepseek-r1:1.5b" on the server to pull the 1.5b model

![x](/static/2025-01-26-ollama-deepseek/04.png)

<br>

And thats pretty much it, now we can interact with this locally run model

![x](/static/2025-01-26-ollama-deepseek/05.png)

<br>

running "/set verbose" we can see the performance that we're getting on this machine, here we get around 6.11 tokens/second, which is not that great

![x](/static/2025-01-26-ollama-deepseek/05a.png)

<br>

## Open WebUI

Now that the deepseek is running on our machine, lets install a front-end framework so we can access it using web browser.
First install docker

![x](/static/2025-01-26-ollama-deepseek/06.png)

<br>

Then open the [Open WebUI github page](https://github.com/open-webui/open-webui) and copy the docker command to pull it onto the local machine

![x](/static/2025-01-26-ollama-deepseek/07.png)

<br>

Paste it on our ubuntu server and let the pulling runs

![x](/static/2025-01-26-ollama-deepseek/08.png)

<br>

To allow ollama to be accessed outside the local machine, add this command to "/etc/systemd/system/ollama.service"

![x](/static/2025-01-26-ollama-deepseek/09.png)

```text
Environment"OLLAMA_HOST=0.0.0.0"
```

<br>

Then restart the ollama service

```text
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

<br>

Access the port 11434 to make sure ollama is running and accessible

![x](/static/2025-01-26-ollama-deepseek/10.png)

<br>

Now we can access Open WebUI on port 3000 and select Deepseek as the model

![x](/static/2025-01-26-ollama-deepseek/11.png)

![x](/static/2025-01-26-ollama-deepseek/12.png)

<br>






















