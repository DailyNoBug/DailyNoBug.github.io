---
title: run-ollama-in-docker
date: 2025-01-15 15:06:31
tags:
---

# Run Ollama in docker

## install nvidia container toolkit

https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#installation

### Installing with Apt

1. Configure the production repository:

   ```
   $ curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
     && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
       sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
       sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
   ```

   Optionally, configure the repository to use experimental packages:

   ```
   $ sed -i -e '/experimental/ s/^#//g' /etc/apt/sources.list.d/nvidia-container-toolkit.list
   ```

2. Update the packages list from the repository:

   ```
   $ sudo apt-get update
   ```

3. Install the NVIDIA Container Toolkit packages:

   ```
   $ sudo apt-get install -y nvidia-container-toolkit
   ```

## Start the container

```bash
docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

## exec the container

```shell
docker exec -it ollama /bin/bash 
```

## webui

可以编写一个flask应用来实现简单的webui：

```python
    from flask import Flask, request, jsonify, render_template_string

    app = Flask(__name__)

    # Ollama API的基本URL
    OLLAMA_API_URL = "https://api.ollama.com/v1/generate"

    @app.route('/')
    def index():
        return render_template_string('''
            <!DOCTYPE html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <title>Chat with Ollama</title>
            </head>
            <body>
                <h1>Welcome to the Chat Interface!</h1>
                <form id="chatForm" method="post" action="/chat">
                    <input type="text" name="message" placeholder="Type your message here...">
                    <button type="submit">Send</button>
                </form>

                <div id="response"></div>

                <script>
                    document.getElementById('chatForm').addEventListener('submit', function(e) {
                        e.preventDefault();
                        const input = document.querySelector('input[name=message]');
                        const xhr = new XMLHttpRequest();
                        xhr.open("POST", "/chat");
                        xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
                        xhr.onload = function() {
                            if (xhr.status === 200) {
                                document.getElementById('response').innerText = xhr.responseText;
                            } else {
                                console.error("Request failed. Returned status of ", xhr.status);
                            }
                        };
                        const formData = new FormData();
                        formData.append("message", input.value);
                        xhr.send(formData);
                    });
                </script>
            </body>
            </html>
        ''')

    @app.route('/chat', methods=['POST'])
    def chat():
        user_input = request.form.get('message')
        
        # 构建API请求参数
        payload = {
            'prompt': f"User: {user_input}\nAssistant:",
            'model': "your_model_name_here",  # 替换为你使用的模型名称
            'max_new_tokens': 100  # 设置生成的最大新令牌数量
        }
        
        try:
            response = requests.post(OLLAMA_API_URL, json=payload)
            response.raise_for_status()
            generated_text = response.json()['choices'][0]['text']
            return jsonify({'response': generated_text})
        except Exception as e:
            return jsonify({'error': str(e)}), 500

    if __name__ == '__main__':
        app.run(debug=True)
```

同时需要html文件：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Chat with Ollama</title>
</head>
<body>
    <h1>Welcome to the Chat Interface!</h1>
    <form id="chatForm" method="post" action="/chat">
        <input type="text" name="message" placeholder="Type your message here...">
        <button type="submit">Send</button>
    </form>

    <div id="response"></div>

    <script>
        document.getElementById('chatForm').addEventListener('submit', function(e) {
            e.preventDefault();
            const input = document.querySelector('input[name=message]');
            const xhr = new XMLHttpRequest();
            xhr.open("POST", "/chat");
            xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
            xhr.onload = function() {
                if (xhr.status === 200) {
                    document.getElementById('response').innerText = xhr.responseText;
                } else {
                    console.error("Request failed. Returned status of ", xhr.status);
                }
            };
            const formData = new FormData();
            formData.append("message", input.value);
            xhr.send(formData);
        });
    </script>
</body>
</html>
```
