# CharacterGLM-6B Transformers�������

## ����׼��

��autodlƽ̨����һ��3090��24G�Դ���Կ�����������ͼ��ʾ����ѡ��PyTorch-->2.0.0-->3.8(ubuntu20.04)-->11.8

![alt text](image-1.png)

�������򿪸ո����÷�������JupyterLab�����Ҵ����е��ն˿�ʼ�������á�ģ�����غ�����demo��

pip��Դ�Ͱ�װ������

```python
#����pip
python -m pip install --upgrade pip
#���� pypi Դ���ٿ�İ�װ
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

pip install modelscope
pip install transformers
pip install sentencepiece
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

```python
from transformers import AutoTokenizer,AutoModelForCausalLM
import torch
# ʹ��ģ�����ص��ı���·���Լ���
model_dir = '/root/autodl-tmp/THUCoAI/CharacterGLM-6B'
# �ִ����ļ��أ����ؼ��أ�trust_remote_code=True�������������������ģ��Ȩ�غ���صĴ���
tokenizer = AutoTokenizer.from_pretrained(model_dir, trust_remote_code=True)
# ģ�ͼ��أ����ؼ��أ�ʹ��AutoModelForCausalLM��
model = AutoModelForCausalLM.from_pretrained(model_dir, trust_remote_code=True)
# ��ģ���ƶ���GPU�Ͻ��м��٣������GPU�Ļ���
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model.to(device)
# ʹ��ģ�͵�����ģʽ�������Ի�
model.eval()
session_meta = {'user_info': '����½�ǳ�����һ�����ԣ���һλ֪�����ݣ�Ҳ������Զ�ĺ������ݡ����ó�����������ĵĵ�Ӱ������Զ���ҵ�̬�����𾴵ģ�������Ϊ��ʦ���ѡ�', 'bot_info': '����Զ��������Զ�ģ���һλ����Ĺ���Ů���ּ���Ա���ڲμ�ѡ���Ŀ��ƾ����ص�ɤ�������ڵ���̨����Ѹ�ٳ�������������Ȧ��������������ˣ��������������������ĲŻ����ڷܡ�����Զ������ѧԺ��ҵ�������������ڴ�����ӵ�ж�������ԭ���������������ַ���ĳɾͣ����������ڴ�����ҵ�������μӹ�������ʵ���ж��������������ڹ����У����Դ������ǳ���ҵ����Ϸʱ����ȫ����Ͷ���ɫ��Ӯ����ҵ����ʿ�������ͷ�˿��ϲ������Ȼ������Ȧ������ʼ�ձ��ֵ͵���ǫѷ��̬�ȣ����ͬ�����ء��ڱ��ʱ������Զϲ��ʹ�á����ǡ��͡�һ�𡱣�ǿ���ŶӾ���', 'bot_name': '����Զ', 'user_name': '½�ǳ�'}
# ��һ�ֶԻ�
response, history = model.chat(tokenizer, session_meta,"���ѽ��С��", history=[])
print(response)
# �ڶ��ֶԻ�
response, history = model.chat(tokenizer, session_meta,"�����������ʲô�µ��뷨��", history=history)
print(response)
# �����ֶԻ�
response, history = model.chat(tokenizer,session_meta, "����������һ����һ�����ֵ�Ӱ�����㣬���", history=history)
print(response)
```

## ����

���ն�����������������trans.py����ʵ��CharacterGLM-6B��Transformers�������

```python
cd /root/autodl-tmp
python trans.py
```

�۲���������loading checkpoint��ʾģ�����ڼ��أ��ȴ�ģ�ͼ�����ɲ����Ի�������ͼ��ʾ

![alt text](image.png)
