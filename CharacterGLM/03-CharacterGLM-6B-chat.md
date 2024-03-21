# CharacterGLM-6B-chat

## ����׼��

��autodlƽ̨����һ��3090��24G�Դ���Կ�����������ͼ��ʾ����ѡ��PyTorch-->2.0.0-->3.8(ubuntu20.04)-->11.8

![alt text](image-1.png)

�������򿪸ո����÷�������JupyterLab�����Ҵ����е��ն˿�ʼ�������á�ģ�����غ�����demo��

pip��Դ�Ͱ�װ������

```python
# ����pip
python -m pip install --upgrade pip
# ���� pypi Դ���ٿ�İ�װ
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

pip install modelscope
pip install transformers
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

����clone���룬��autodlƽ̨�Դ���ѧ��������١�ѧ�����������ϸʹ���뿴��
https://www.autodl.com/docs/network_turbo/

```python
source /etc/network_turbo
```

Ȼ���л�·��, clone����.

```python
cd /root/autodl-tmp
git clone https://github.com/thu-coai/CharacterGLM-6B
```

## demo����

�޸Ĵ���·������ /root/autodl-tmp/CharacterGLM-6B/basic_demo/web_demo_streamlit.py�е�20�е�ģ�͸���Ϊ���ص�/root/autodl-tmp/THUCoAI/CharacterGLM-6B

![alt text](../image/03-�޸�·��.png)

�޸�requirements.txt�ļ��������е�torchɾ�����������Ѿ�����torch������Ҫ�ٰ�װ��Ȼ��ִ����������

```python
cd /root/autodl-tmp/CharacterGLM-6B
pip install -r requirements.txt
```

���ն�������������������������,����cd��basic_demo�ļ����£���ֹ�Ҳ���character.json�ļ�

```python
cd /root/autodl-tmp/CharacterGLM-6B/basic_demo
streamlit run ./web_demo2.py --server.address 127.0.0.1 --server.port 6006
```

![alt text](../image/03-����webdemo.png)

�ڽ� autodl �Ķ˿�ӳ�䵽���ص� http://localhost:6006 �󣬼��ɿ���demo���档����ӳ�䲽��ο��ĵ�General-Setting�ļ�����/02-AutoDL���Ŷ˿�.md�ĵ���

��������� http://localhost:6006 ���棬ģ�ͼ��أ�����ʹ�ã�����ͼ��ʾ��

![alt text](../image/03-webdemo_show.png)

## ����������

�޸Ĵ���·������ /root/autodl-tmp/CharacterGLM-6B/basic_demo/cli_demo.py�е�20�е�ģ�͸���Ϊ���ص�/root/autodl-tmp/THUCoAI/CharacterGLM-6B

���ն�������������������������

```python
cd /root/autodl-tmp/CharacterGLM-6B/basic_demo
python ./cli_demo.py 
```

![alt text](../image/03-����clidemo.png)