>   
> OLLama让那些没有GPU，还想玩一玩大语言模型成为可能，普通的CPU也可以跑个千问，gemma。ollama有自己python包，直接通过使用包来实现本地ollama的的所有功能，另外也提供api，调用方法也很简单，与openai调用方法类似，叫兼容openai接口。ollama的api文档，通过调用api，可以查询本地模型列表，进行聊天对话，聊天补全，创建模型，拉取，删除模型等操作。最后用python做一个翻译助手演示。

## 1.生成补全

#### 格式

POST /api/generate   

使用提供的模型生成给定提示的响应。这是一个流端点，因此会有一系列响应。最终响应对象将包括来自请求的统计信息和附加数据。

#### 参数

- `model`：（必填）模型名称，后面可以跟`tag`。比如`gemma:7b`
    
- `prompt`：生成响应的提示
    
- `images`：（可选）base64 编码图像的列表（对于多模式模型，例如`llava`）
    

#### 高级参数（可选）：

- `format`：返回响应的格式。目前唯一接受的值是`json`
    
- `options`：模型文件文档中列出的其他模型参数，例如`temperature`
    
- `system`：系统消息（覆盖 中定义的内容`Modelfile`）
    
- `template`：要使用的提示模板（覆盖 中定义的内容`Modelfile`）
    
- `context`：从先前的请求返回的上下文参数`/generate`，这可用于保留简短的会话记忆
    
- `stream`：`false`响应是否作为单个响应对象返回，而不是对象流
    
- `raw`：如果`true`没有格式化，将应用于提示。`raw`如果您在 API 请求中指定完整模板化提示，则可以选择使用该参数
    
- `keep_alive`：控制模型在请求后加载到内存中的时间（默认值`5m`：）
    

请求和响应的格式均为`json`格式，发出请求的格式：

curl http://localhost:11434/api/generate -d '{     "model": "llama2",     "prompt": "What color is the sky at different times of the day? Respond using JSON",     "format": "json",     "stream": false   }'   

高级参数都可以在请求中携带，比如`keep_alive`，默认是5分钟，5分钟内没有任何操作，释放内存。如果是`-1`，是一直加载在内存。![](https://api.ibos.cn/v4/weapparticle/accesswximg?aid=79366&url=aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9IYkFrSWlhc3dpY0s3cVJRYjlOaWFERFEweU1pYnZla3A2UXJJWGxFMGliMjNpY2dVRmRpY0M1TGZmOUJpYTFDbGZSc1dMeHpESjkwZ0QxajBzYUlvN0NCNHpwazJnLzY0MD93eF9mbXQ9cG5nJmFtcA==;from=appmsg)

响应返回的格式：

{     "model": "llama2",     "created_at": "2023-11-09T21:07:55.186497Z",     "response": "{\n\"morning\": {\n\"color\": \"blue\"\n},\n\"noon\": {\n\"color\": \"blue-gray\"\n},\n\"afternoon\": {\n\"color\": \"warm gray\"\n},\n\"evening\": {\n\"color\": \"orange\"\n}\n}\n",     "done": true,     "context": [1, 2, 3],     "total_duration": 4648158584,     "load_duration": 4071084,     "prompt_eval_count": 36,     "prompt_eval_duration": 439038000,     "eval_count": 180,     "eval_duration": 4196918000   }   

在 `powershell`访问`API`格式为：

(Invoke-WebRequest -method POST -Body '{"model":"llama2", "prompt":"Why is the sky blue?", "stream": false}' -uri http://localhost:11434/api/generate ).Content | ConvertFrom-json   

`python`访问`API`：

url_generate = "http://localhost:11434/api/generate"   def get_response(url, data):       response = requests.post(url, json=data)       response_dict = json.loads(response.text)       response_content = response_dict["response"]       return response_content      data = {       "model": "gemma:7b",       "prompt": "Why is the sky blue?",       "stream": False   }         res = get_response(url_generate,data)   print(res)   

上面是通过`python`对接口进行访问，可在程序代码直接调用，适合批量操作，生成结果。

正常请求时，`options`都省略了，`options`可以设置很多参数，比如`temperature`，是否使用`gpu`，上下文的长度等，都在此设置。下面是一个包含`options`的请求：

curl http://localhost:11434/api/generate -d '{     "model": "llama2",     "prompt": "Why is the sky blue?",     "stream": false,     "options": {       "num_keep": 5,       "seed": 42,       "num_predict": 100,       "top_k": 20,       "top_p": 0.9,       "tfs_z": 0.5,       "typical_p": 0.7,       "repeat_last_n": 33,       "temperature": 0.8,       "repeat_penalty": 1.2,       "presence_penalty": 1.5,       "frequency_penalty": 1.0,       "mirostat": 1,       "mirostat_tau": 0.8,       "mirostat_eta": 0.6,       "penalize_newline": true,       "stop": ["\n", "user:"],       "numa": false,       "num_ctx": 1024,       "num_batch": 2,       "num_gqa": 1,       "num_gpu": 1,       "main_gpu": 0,       "low_vram": false,       "f16_kv": true,       "vocab_only": false,       "use_mmap": true,       "use_mlock": false,       "rope_frequency_base": 1.1,       "rope_frequency_scale": 0.8,       "num_thread": 8     }   }'   

## 2.生成聊天补全

#### 格式

POST /api/chat   

和上面生成补全很像。

#### 参数

- `model`：（必填）型号名称
    
- `messages`：聊天的消息，这个可以用来保留聊天记忆
    

该`message`对象具有以下字段：

- `role`：消息的角色，`system`或者`user` `assistant`
    
- `content`: 消息内容
    
- `images`（可选）：要包含在消息中的图像列表（对于多模式模型，例如`llava`）
    

#### 高级参数（可选）：

- `format`：返回响应的格式。目前唯一接受的值是`json`
    
- `options`：模型文件文档中列出的其他模型参数，例如`temperature`
    
- `template`：要使用的提示模板（覆盖 中定义的内容`Modelfile`）
    
- `stream`：`false`响应是否作为单个响应对象返回，而不是对象流
    
- `keep_alive`：控制模型在请求后加载到内存中的时间（默认值`5m`：）
    

发送聊天请求：

curl http://localhost:11434/api/chat -d '{     "model": "llama2",     "messages": [       {         "role": "user",         "content": "why is the sky blue?"       }     ]   }'   

和`generate`的区别，`message`和`prompt`对应，`prompt`后面直接跟要聊的内容，而`message`里面还有`role`角色，`user`相当于提问的内容。

响应返回的内容：

{     "model": "llama2",     "created_at": "2023-08-04T19:22:45.499127Z",     "done": true,     "total_duration": 4883583458,     "load_duration": 1334875,     "prompt_eval_count": 26,     "prompt_eval_duration": 342546000,     "eval_count": 282,     "eval_duration": 4535599000   }   

还可以发送带聊天记录的请求：

curl http://localhost:11434/api/chat -d '{     "model": "llama2",     "messages": [       {         "role": "user",         "content": "why is the sky blue?"       },       {         "role": "assistant",         "content": "due to rayleigh scattering."       },       {         "role": "user",         "content": "how is that different than mie scattering?"       }     ]   }'   

`python`格式的生成聊天补全：

url_chat =  "http://localhost:11434/api/chat"   data = {       "model": "llama2",       "messages": [       {         "role": "user",         "content": "why is the sky blue?"       },       "stream": False   }   response = requests.post(url_chat, json=data)   response_dict = json.loads(response.text)   print(response_dict)   

## 3.创建模型

#### 格式

POST /api/create   

#### 参数

- `name`：要创建的模型的名称
    
- `modelfile`（可选）：模型文件的内容
    
- `stream`：（可选）如果`false`响应将作为单个响应对象返回，而不是对象流
    
- `path`（可选）：模型文件的路径`modelfile`后面直接是`modelfile`的内容，比如基于那个模型，有那些设定，
    

创建模型的请求：

curl http://localhost:11434/api/create -d '{     "name": "mario",     "modelfile": "FROM llama2\nSYSTEM You are mario from Super Mario Bros."   }'   

基于`llama2`创建一个模型，系统角色进行设定。返回结果就不多做介绍。

使用`python`创建一个模型：

url_create =  "http://localhost:11434/api/create"   data = {    "name": "mario",    "modelfile": "FROM llama2\nSYSTEM You are mario from Super Mario Bros."   }   response = requests.post(url, json=data)   response_dict = json.loads(response.text)   print(response_dict)   

这个`python`和上面的相同的功能。

## 4.显示模型

#### 格式

GET /api/tags   

列出本地所有模型。![](https://api.ibos.cn/v4/weapparticle/accesswximg?aid=79366&url=aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9IYkFrSWlhc3dpY0s3cVJRYjlOaWFERFEweU1pYnZla3A2UXJQV1RscWE3RmxSeFlpY3dRazRNOXdGV0c5YlE4c2xpY0JqdlBwVHZUbmtQTjBObkNuaWJFWW5MNFEvNjQwP3d4X2ZtdD1wbmcmYW1w;from=appmsg)

使用`python`显示模型。

url_list = "http://localhost:11434/api/tags"   def get_list(url):       response = requests.get(url)       response_dict = json.loads(response.text)       model_names = [model["name"] for model in response_dict["models"]]       names = []       # 打印所有模型的名称       for name in model_names:           names.append(name)       for idx, name in enumerate(names, start=1):           print(f"{idx}. {name}")       return names   get_list(url_list)   

返回结果：

1. codellama:13b   2. codellama:7b-code   3. gemma:2b   4. gemma:7b   5. gemma_7b:latest   6. gemma_sumary:latest   7. llama2:7b   8. llama2:latest   9. llava:7b   10. llava:v1.6   11. mistral:latest   12. mistrallite:latest   13. nomic-embed-text:latest   14. qwen:1.8b   15. qwen:4b   16. qwen:7b   

## 5.显示模型信息

#### 格式

POST /api/show   

显示有关模型的信息，包括详细信息、模型文件、模板、参数、许可证和系统提示。

#### 参数

- `name`：要显示的模型名称
    

#### 请求

curl http://localhost:11434/api/show -d '{     "name": "llama2"   }'   

使用`python`显示模型信息：

url_show_info  = "http://localhost:11434/api/show"   def show_model_info(url,model_name):       data = {           "name": model_name               }       response = requests.post(url, json=data)       response_dict = json.loads(response.text)       print(response_dict)   show_model_info(url_show_info,"gemma:7b")   

返回的结果：

{'license': 'Gemma Terms of Use \n\nLast modified: February 21, 2024\n\nBy using, reproducing, modifying, distributing, performing or displaying any portion or element of Gemma, Model Derivatives including via any Hosted Service, (each as defined below) (collectively, the "Gemma Services") or otherwise accepting the terms of this Agreement, you agree to be bound by this Agreement.\n\nSection 1: DEFINITIONS\n1.1 Definitions\n(a) "Agreement" or "Gemma Terms of Use" means these terms and conditions that govern the use, reproduction, Distribution or modification of the Gemma Services and any terms and conditions incorporated by reference.\n\n(b) "Distribution" or "Distribute" means any transmission, publication, or other sharing of Gemma or Model Derivatives to a third party, including by providing or making Gemma or its functionality available as a hosted service via API, web access, or any other electronic or remote means ("Hosted Service").\n\n(c) "Gemma" means the set of machine learning language models, trained model weights and parameters identified at ai.google.dev/gemma, regardless of the source that you obtained it from.\n\n(d) "Google" means Google LLC.\n\n(e) "Model Derivatives" means all (i) modifications to Gemma, (ii) works based on Gemma, or (iii) any other machine learning model which is created by transfer of patterns of the weights, parameters, operations, or Output of Gemma, to that model in order to cause that model to perform similarly to Gemma, including distillation methods that use intermediate data representations or methods based on the generation of synthetic data Outputs by Gemma for training that model. For clarity, Outputs are not deemed Model Derivatives.\n\n(f) "Output" means the information content output of Gemma or a Model Derivative that results from operating or otherwise using Gemma or the Model Derivative, including via a Hosted Service.\n\n1.2\nAs used in this Agreement, "including" means "including without limitation".\n\nSection 2: ELIGIBILITY AND USAGE\n2.1 Eligibility\nYou represent and warrant that you have the legal capacity to enter into this Agreement (including being of sufficient age of consent). If you are accessing or using any of the Gemma Services for or on behalf of a legal entity, (a) you are entering into this Agreement on behalf of yourself and that legal entity, (b) you represent and warrant that you have the authority to act on behalf of and bind that entity to this Agreement and (c) references to "you" or "your" in the remainder of this Agreement refers to both you (as an individual) and that entity.\n\n2.2 Use\nYou may use, reproduce, modify, Distribute, perform or display any of the Gemma Services only in accordance with the terms of this Agreement, and must not violate (or encourage or permit anyone else to violate) any term of this Agreement.\n\nSection 3: DISTRIBUTION AND RESTRICTIONS\n3.1 Distribution and Redistribution\nYou may reproduce or Distribute copies of Gemma or Model Derivatives if you meet all of the following conditions:\n\nYou must include the use restrictions referenced in Section 3.2 as an enforceable provision in any    .......   

## 6.其他

除了以上功能，还可以复制模型，删除模型，拉取模型，另外，如果有ollama的帐号，还可把模型推到ollama的服务器。

## 7.OLLama相关设置

### 1.本地模型存储

windows用户默认存储位置：

C:\Users\<username>\.ollama\models   

更改默认存储位置，在环境变量中设置`OLLAMA_MODELS`对应存储位置，实现模型存储位置更改。

### 2.导入GGUF模型

可能有从`HuggingFace`下载的`gguf`模型，可以通过`modelfile`创建模型导入`gguf`模型。创建一个`Modelfile`文件：

FROM ./mistral-7b-v0.1.Q4_0.gguf   

通过这个Modelfile创建新模型：

ollama create example -f Modelfile   

`example`为新模型名，使用时直接调用这个模型名就可以。

### 3.参数设置

正常运行模型时，很少对参数进行设置，在发送请求时，可以通过`options`对参数进行设置，比如设置上下文的`token`数：

curl http://localhost:11434/api/generate -d '{     "model": "llama2",     "prompt": "Why is the sky blue?",     "options": {       "num_ctx": 4096     }   }'   

默认是2048，这里修改成了4096，还可以设置比如是否使用gpu，后台服务跑起来，刚出来这些东西，都可以在参数里进行设置。![](https://api.ibos.cn/v4/weapparticle/accesswximg?aid=79366&url=aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9IYkFrSWlhc3dpY0s3cVJRYjlOaWFERFEweU1pYnZla3A2UXJ1YTdlc3RvWVhMTGh2dnNyeVJGOE1pYkZJaWJKQkpEazd5Q3doY0RFNEF0elpWdzBZenNLeTRCQS82NDA/d3hfZm10PXBuZyZhbXA=;from=appmsg)

## 8.兼容openai

兼容`openai`接口，通过`openai`的包可以直接调用访问`ollama`提供的后台服务。

from openai import OpenAI   client = OpenAI(       base_url='http://localhost:11434/v1/',       # required but ignored       api_key='ollama',   )      chat_completion = client.chat.completions.create(       messages=[           {               'role': 'user',               'content': 'Say this is a test',           }       ],       model='llama2',   )   

得到：

ChatCompletion(id='chatcmpl-173', choices=[Choice(finish_reason='stop', index=0, logprobs=None, message=ChatCompletionMessage(content='\nThe question " Why is the sky blue? " is a common one, and there are several reasons why the sky appears blue to our eyes. Here are some possible explanations:\n\n1. Rayleigh scattering: When sunlight enters Earth\'s atmosphere, it encounters tiny molecules of gases such as nitrogen and oxygen. These molecules scatter the light in all directions, but they scatter shorter (blue) wavelengths more than longer (red) wavelengths. This is known as Rayleigh scattering. As a result, the blue light is dispersed throughout the atmosphere, giving the sky its blue appearance.\n2. Mie scattering: In addition to Rayleigh scattering, there is also a phenomenon called Mie scattering, which occurs when light encounters much larger particles in the atmosphere, such as dust and water droplets. These particles can also scatter light, but they preferentially scatter longer (red) wavelengths, which can make the sky appear more red or orange during sunrise and sunset.\n3. Angel\'s breath: Another explanation for why the sky appears blue is due to a phenomenon called "angel\'s breath." This occurs when sunlight passes through a layer of cool air near the Earth\'s surface, causing the light to be scattered in all directions and take on a bluish hue.\n4. Optical properties of the atmosphere: The atmosphere has its own optical properties, which can affect how light is transmitted and scattered. For example, the atmosphere scatters shorter wavelengths (such as blue and violet) more than longer wavelengths (such as red and orange), which can contribute to the blue color of the sky.\n5. Perspective: The way we perceive the color of the sky can also be affected by perspective. From a distance, the sky may appear blue because our brains are wired to perceive blue as a color that is further away. This is known as the "Perspective Problem."\n\nIt\'s worth noting that the color of the sky can vary depending on the time of day, the amount of sunlight, and other environmental factors. For example, during sunrise and sunset, the sky may appear more red or orange due to the scattering of light by atmospheric particles.', role='assistant', function_call=None, tool_calls=None))], created=1710810193, model='llama2:7b', object='chat.completion', system_fingerprint='fp_ollama', usage=CompletionUsage(completion_tokens=498, prompt_tokens=34, total_tokens=532))   

## 9.翻译助手

最后一个实现翻译助手，这么多大模型，中西语料足够，让他充当个免费翻译没问题吧。我愿意在网上找英文资源，有时会没有字幕，自己英语又不好，如果能把字幕翻译的活干好了，这个大模型学习，也算有所收获。下面通过`python`代码，访问`ollama`，给他设定一个身份，让他充当一个翻译的角色，后面只给他英文内容，他直接输出中文内容（"Translate the following into chinese and only show me the translated"）。只是一个`demo`，字幕提取，读取翻译应该都可以搞定。下面演示是要翻译的内容为`grok`网页介绍内容，看一下他翻译的效果。

import requests   import json      text = """   We are releasing the base model weights and network architecture of Grok-1, our large language model. Grok-1 is a 314 billion parameter Mixture-of-Experts model trained from scratch by xAI.      This is the raw base model checkpoint from the Grok-1 pre-training phase, which concluded in October 2023. This means that the model is not fine-tuned for any specific application, such as dialogue.      We are releasing the weights and the architecture under the Apache 2.0 license.      To get started with using the model, follow the instructions at github.com/xai-org/grok.      Model Details   Base model trained on a large amount of text data, not fine-tuned for any particular task.   314B parameter Mixture-of-Experts model with 25% of the weights active on a given token.   Trained from scratch by xAI using a custom training stack on top of JAX and Rust in October 2023.   The cover image was generated using Midjourney based on the following prompt proposed by Grok: A 3D illustration of a neural network, with transparent nodes and glowing connections, showcasing the varying weights as different thicknesses and colors of the connecting lines.   """         #"Describe the bug. When selecting to use a self hosted ollama instance, there is no way to do 2 things:Set the server endpoint for the ollama instance. in my case I have a desktop machine with a good GPU and run ollama there, when coding on my laptop i want to use the ollama instance on my desktop, no matter what value is set for cody.autocomplete.advanced.serverEndpoint, cody will always attempt to use http://localhost:11434, so i cannot sepcify the ip of my desktop machine hosting ollama.Use a different model on ollama - no matter what value is set for cody.autocomplete.advanced.model, for example when llama-code-13b is selected, the vscode output tab for cody always says: █ CodyCompletionProvider:initialized: unstable-ollama/codellama:7b-code "      url_generate = "http://localhost:11434/api/generate"      data = {       "model": "mistral:latest",       "prompt": f"{text}",#"Why is the sky blue?",       "system":"Translate the following into chinese and only show me the translated",       "stream": False   }      def get_response(url, data):       response = requests.post(url, json=data)       response_dict = json.loads(response.text)       response_content = response_dict["response"]       return response_content      res = get_response(url_generate,data)   print(res)   

大概演示一下，具体细节再调整吧。今天内容到些结束。