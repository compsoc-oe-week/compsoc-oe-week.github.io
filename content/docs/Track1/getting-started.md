---
title: "Getting Started with Track 1"
---

# Overview

For Track 1, we've allocated 2 shared vLLM instances to be used for your solutions in the challenge. These instances are OpenAI API compatible, meaning they can be interacted with tooling and libraries originally created for working with OpenAI.

The first instance can be located at `eidf219-main.vms.os.eidf.epcc.ed.ac.uk:8000` and is a `Qwen/Qwen3-Coder-480B-A35B-Instruct-FP8` model under the hood, to be used for general purpose queries and code generation. The second instance is a visual reasoning focused model and can be found at `eidf219-main.vms.os.eidf.epcc.ed.ac.uk:8001` (a `Qwen/Qwen2.5-VL-32B-Instruct` model under the hood).

# Step 1: Connecting to the API

## Connecting from your team's VM

The first and easiest way to connect to the API is to connect to your team's VM by following the instructions found [here](). From there, you can run a test query against the vLLM instances like so:
```bash
curl http://eidf219-main.vms.os.eidf.epcc.ed.ac.uk:8000/v1/chat/completions -H "Content-Type: application/json" -d '{
  "model": "Qwen/Qwen3-Coder-480B-A35B-Instruct-FP8",
  "messages": [
    {"role": "user", "content": "Give me a short introduction to large language models."}
  ],
  "temperature": 0.6,
  "top_p": 0.95,
  "top_k": 20,
  "max_tokens": 32768
}'
```

## Connecting from your local machine with SSH tunelling

To connect to the vLLM instances from your local machine, you can use SSH tunneling. In a standard bash shell, the command looks like so,
```
ssh -J <your_username>@eidf-gateway.epcc.ed.ac.uk -L 8000:10.1.0.155:8000 -N <your_username>@<vm_ip>
```
where `<your_username>` is replaced with the username given for your team's VM, and `<vm_ip>` is the EIDF IP address associated with your team's VM. Note: when connecting to the visual reasoning vLLM instance (the one using a `Qwen/Qwen2.5-VL-32B-Instruct` mode), replace `8000:10.1.0.155:8000` with `8001:10.1.0.155:8001`.


Once you've done this, in a new window you should be able to run a similar test, now by querying the API as `localhost:8000` (or `localhost:8001` if you're querying the visual reasoning instance). 
```
curl http://localhost:8000/v1/chat/completions -H "Content-Type: application/json" -d '{
  "model": "Qwen/Qwen3-Coder-480B-A35B-Instruct-FP8",
  "messages": [
    {"role": "user", "content": "Give me a short introduction to large language models."}
  ],
  "temperature": 0.6,
  "top_p": 0.95,
  "top_k": 20,
  "max_tokens": 32768
}'
```

**If you're trying to connect from a docker container, you can add the argument `--network=host` to be able to connect to localhost from inside the docker container on linux, or connect to the `host.docker.internal:8000` endpoint instead.**.

# Step 2: Programmatically interacting with the API

In Python, the `openai` package can be used to interact with the vLLM instance. Under the hood, this is largely a wrapper around the OpenAI REST API, so the insights from the previous sections still apply. I've included a few code samples to get you started with these below.

To query the `Qwen/Qwen3-Coder-480B-A35B-Instruct-FP8` model using the `openai` package:
```python
from openai import OpenAI

openai_api_key = "EMPTY" # This can be left alone

# Uncomment depending on where you're running this script:
# openai_api_base = "http://localhost:8000/v1" # I'm running from my local machine with SSH tunneling
# openai_api_base = "http://eidf219-main.vms.os.eidf.epcc.ed.ac.uk:8000/v1" # I'm running from my team's VM

client = OpenAI(
    api_key=openai_api_key,
    base_url=openai_api_base,
)

chat_response = client.chat.completions.create(
    model="Qwen/Qwen3-Coder-480B-A35B-Instruct-FP8",
    messages=[
        {"role": "user", "content": "Give me a short introduction to large language models."},
    ],
    max_tokens=32768,
    temperature=0.6,
    top_p=0.95,
    extra_body={
        "top_k": 20,
    },
)
print("Chat response:", chat_response)
```

To query the `Qwen/Qwen2.5-VL-32B-Instruct` model using REST API requests:
```python
import requests
import base64

# Uncomment depending on where you're running this script:
# openai_api_base = "http://localhost:8001/v1" # I'm running from my local machine with SSH tunneling
# openai_api_base = "http://eidf219-main.vms.os.eidf.epcc.ed.ac.uk:8001/v1" # I'm running from my team's VM


# Replace this with your actual image URL
image_url = "https://dashscope.oss-cn-beijing.aliyuncs.com/images/dog_and_girl.jpeg"

# Download the image and encode it as base64
image_data = requests.get(image_url).content
image_base64 = base64.b64encode(image_data).decode('utf-8')

# Prepare the payload for the OpenAI-compatible API
payload = {
    "model": "Qwen/Qwen2.5-VL-32B-Instruct",
    "messages": [
        {
            "role": "user",
            "content": [
                {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{image_base64}"}},
                {"type": "text", "text": "What is inside this image?"}
            ]
        }
    ]
}

# Send the request to the local server
response = requests.post(f"{openai_api_base}/chat/completions", json=payload)

# Print the model's response
print(response.json())
```
