# CharacterGLM-6B FastApi�������

## ����׼��

��autodlƽ̨����һ��3090��24G�Դ���Կ�����������ͼ��ʾ����ѡ��PyTorch-->2.0.0-->3.8(ubuntu20.04)-->11.8

![alt text](image-1-1.png)

�������򿪸ո����÷�������JupyterLab�����Ҵ����е��ն˿�ʼ�������á�ģ�����غ�����demo��

pip��Դ�Ͱ�װ������

```python
# ����pip
python -m pip install --upgrade pip
# ���� pypi Դ���ٿ�İ�װ
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

pip install fastapi==0.104.1
pip install uvicorn==0.24.0.post1
pip install requests==2.25.1
pip install modelscope==1.9.5
pip install transformers==4.37.2
pip install streamlit==1.24.0
pip install sentencepiece==0.1.99
pip install accelerate==0.24.1

```

## ģ������

ʹ�� modelscope �е�snapshot_download��������ģ�ͣ���һ������Ϊģ�����ƣ�����cache_dirΪģ�͵�����·����

�� /root/autodl-tmp ·�����½� download.py �ļ��������������������ݣ�ճ�������ǵñ����ļ�������ͼ��ʾ�������� python /root/autodl-tmp/download.pyִ�����أ�ģ�ʹ�СΪ 12 GB������ģ�ʹ����Ҫ 10~15 ����

```python
import torch
from modelscope import snapshot_download, AutoModel, AutoTokenizer
import os
model_dir = snapshot_download('THUCoAI/CharacterGLM-6B', cache_dir='/root/autodl-tmp', revision='master')
```

## ����׼��

��/root/autodl-tmp·�����½�api.py�ļ��������������������ݣ�ճ�������ǵñ����ļ�������Ĵ����к���ϸ��ע�ͣ�������в����ĵط�����ӭ���issue��

```python
from fastapi import FastAPI, Request
from transformers import AutoTokenizer, AutoModelForCausalLM
import uvicorn
import json
import datetime
import torch

# �����豸����
DEVICE = "cuda"  # ʹ��CUDA
DEVICE_ID = "0"  # CUDA�豸ID�����δ������Ϊ��
CUDA_DEVICE = f"{DEVICE}:{DEVICE_ID}" if DEVICE_ID else DEVICE  # ���CUDA�豸��Ϣ

# ����GPU�ڴ溯��
def torch_gc():
    if torch.cuda.is_available():  # ����Ƿ����CUDA
        with torch.cuda.device(CUDA_DEVICE):  # ָ��CUDA�豸
            torch.cuda.empty_cache()  # ���CUDA����
            torch.cuda.ipc_collect()  # �ռ�CUDA�ڴ���Ƭ

# ����FastAPIӦ��
app = FastAPI()

# ����POST����Ķ˵�
@app.post("/")
async def create_item(request: Request):
    global model, tokenizer  # ����ȫ�ֱ����Ա��ں����ڲ�ʹ��ģ�ͺͷִ���
    json_post_raw = await request.json()  # ��ȡPOST�����JSON����
    json_post = json.dumps(json_post_raw)  # ��JSON����ת��Ϊ�ַ���
    json_post_list = json.loads(json_post)  # ���ַ���ת��ΪPython����
    prompt = json_post_list.get('prompt')  # ��ȡ�����е���ʾ
    history = json_post_list.get('history')  # ��ȡ�����е���ʷ��¼
    max_length = json_post_list.get('max_length')  # ��ȡ�����е���󳤶�
    top_p = json_post_list.get('top_p')  # ��ȡ�����е�top_p����
    temperature = json_post_list.get('temperature')  # ��ȡ�����е��¶Ȳ���
    session_meta = {'user_info': '����½�ǳ�����һ�����ԣ���һλ֪�����ݣ�Ҳ������Զ�ĺ������ݡ����ó�����������ĵĵ�Ӱ������Զ���ҵ�̬�����𾴵ģ�������Ϊ��ʦ���ѡ�', 'bot_info': '����Զ��������Զ�ģ���һλ����Ĺ���Ů���ּ���Ա���ڲμ�ѡ���Ŀ��ƾ����ص�ɤ�������ڵ���̨����Ѹ�ٳ�������������Ȧ��������������ˣ��������������������ĲŻ����ڷܡ�����Զ������ѧԺ��ҵ�������������ڴ�����ӵ�ж�������ԭ���������������ַ���ĳɾͣ����������ڴ�����ҵ�������μӹ�������ʵ���ж��������������ڹ����У����Դ������ǳ���ҵ����Ϸʱ����ȫ����Ͷ���ɫ��Ӯ����ҵ����ʿ�������ͷ�˿��ϲ������Ȼ������Ȧ������ʼ�ձ��ֵ͵���ǫѷ��̬�ȣ����ͬ�����ء��ڱ��ʱ������Զϲ��ʹ�á����ǡ��͡�һ�𡱣�ǿ���ŶӾ���', 'bot_name': '����Զ', 'user_name': '½�ǳ�'}
    # ����ģ�ͽ��жԻ�����
    response, history = model.chat(
        tokenizer,
        session_meta,
        prompt,
        history=history,
        max_length=max_length if max_length else 2048,  # ���δ�ṩ��󳤶ȣ�Ĭ��ʹ��2048
        top_p=top_p if top_p else 0.7,  # ���δ�ṩtop_p������Ĭ��ʹ��0.7
        temperature=temperature if temperature else 0.95  # ���δ�ṩ�¶Ȳ�����Ĭ��ʹ��0.95
    )
    now = datetime.datetime.now()  # ��ȡ��ǰʱ��
    time = now.strftime("%Y-%m-%d %H:%M:%S")  # ��ʽ��ʱ��Ϊ�ַ���
    # ������ӦJSON
    answer = {
        "response": response,
        "history": history,
        "status": 200,
        "time": time
    }
    # ������־��Ϣ
    log = "[" + time + "] " + '", prompt:"' + prompt + '", response:"' + repr(response) + '"'
    print(log)  # ��ӡ��־
    torch_gc()  # ִ��GPU�ڴ�����
    return answer  # ������Ӧ

# ���������
if __name__ == '__main__':
    # ����Ԥѵ���ķִ�����ģ��
    tokenizer = AutoTokenizer.from_pretrained("/root/autodl-tmp/THUCoAI/CharacterGLM-6B", trust_remote_code=True)
    model = AutoModelForCausalLM.from_pretrained("/root/autodl-tmp/THUCoAI/CharacterGLM-6B", trust_remote_code=True).to(torch.bfloat16).cuda()
    model.eval()  # ����ģ��Ϊ����ģʽ
    # ����FastAPIӦ��
    # ��6006�˿ڿ��Խ�autodl�Ķ˿�ӳ�䵽���أ��Ӷ��ڱ���ʹ��api
    uvicorn.run(app, host='0.0.0.0', port=6006, workers=1)  # ��ָ���˿ں�����������Ӧ��
```

## Api�������

���ն�����������������api����

```python

cd /root/autodl-tmp
python api.py

```

Ĭ�ϲ����� 6006 �˿ڣ�ͨ�� POST �������е��ã�����ʹ��curl���ã�������ʾ��

```python

curl -X POST "http://127.0.0.1:6006" \
     -H 'Content-Type: application/json' \
     -d '{"prompt": "���", "history": []}'

```

����ʾ���������ͼ��ʾ

![alt text](a3367af175859d83496a0357d56daa6.png)

Ҳ����ʹ��python�е�requests����е��ã��½�api-requests.py�ļ���д�����´��룺

```python

import requests
import json

def get_completion(prompt):
    headers = {'Content-Type': 'application/json'}
    data = {"prompt": prompt, "history": []}
    response = requests.post(url='http://127.0.0.1:6006', headers=headers, data=json.dumps(data))
    return response.json()['response']

if __name__ == '__main__':
    print(get_completion('����˭ѽ��'))

```

�¿�һ���նˣ���������ָ��

```python
cd /root/autodl-tmp
python api-requests.py

```

�õ��ķ���ֵ�����չʾ����

```python
{
'response': '��,��ã��ҽ�����Զ����΢Ц����Է���ȥ��', 
'history': [['����˭ѽ��', '��,��ã��ҽ�����Զ����΢Ц����Է���ȥ��']], 
'status': 200, 
'time': '2024-03-05 22:44:35'
}
```

![alt text](1709650011260.png)
