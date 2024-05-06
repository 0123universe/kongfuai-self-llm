# 04-CharacterGLM-6B-Chat Lora΢��

## ����

���ļ�Ҫ������λ���transformers��peft�ȿ�ܣ���CharacterGLM-6B-chatģ�ͽ���Lora΢����Loraԭ��ɲο����ͣ�֪��|����ǳ��Lora

## ��������

����ɻ����������úͱ���ģ�Ͳ��������£�����Ҫ��װһЩ�������⣬����ʹ���������

```python
pip install transformers==4.37.2
pip install peft==0.4.0.dev0
pip install datasets==2.10.1
pip install accelerate==0.21.0

```

�ڱ��������У���΢�����ݼ������ڸ�Ŀ¼/dataset��

## ָ�����

LLM΢��һ��ָָ��΢�����̡���νָ��΢������˵����ʹ�õ�΢���������磺

```python
{
    "instruction":"�ش��û��������⣬ֱ�Ӹ��������"
    "input":"�й���һ��ŵ������������˭��"
    "output":"Ī��"
}
```

����instruction���û�ָ���֪ģ����Ҫ��ɵ�����input���û����룬������û�ָ����������������ݣ�output��ģ��Ӧ�ø����������

�����ǵĺ���ѵ��Ŀ������ģ�;�����Ⲣ��ѭ�û�ָ�����������ˣ���ָ�����ʱ������Ӧ������ǵ�Ŀ����������Թ�������ָ����ڱ�������ʹ���ɱ��ߺ�����Դ��Chat-�����Ŀ��Ϊʾ�������ǵ�Ŀ���ǹ���һ���ܹ�ģ����ֶԻ����ĸ��Ի�LLM��������ǹ�����ָ�����磺

```python
{
    "instruction": "",
    "input":"����˭��",
    "output":"�Ҹ��Ǵ�����������Զ����"
}
```

���ǹ����ȫ��ָ�����ݼ��ڸ�Ŀ¼�¡�

## QA��Instruction���������ϵ

QA��ָһ��һ�����ʽ��ͨ�����û����ʣ�ģ�͸����ش𡣶�instruction��Դ����Prompt Engineering���������ֳ��������֣�Instruction������������Input��������������Ķ���

�ʴ�(QA)��ʽ��ѵ������ͨ������ѵ��ģ��ִ�о����������磬�������⡰�����INFJ��ENTP����MBTI�Ը�֮�������

*�ʴ�(QA)��ʽ��

```python
ָ��(instruction)��
����(input)��INFJ��ENTP������MBTI�Ը�֮���������ʲô��
```

*ָ��(Instruction)��ʽ��

```python
ָ��(Instruction):�������������MBTI�Ը������
����(input):INFJ��ENTP
```

## ���ݸ�ʽ��

Loraѵ������������Ҫ������ʽ��������֮���������ģ�ͽ���ѵ���ģ�����һ����Ҫ�������ı�����Ϊinput_ids��������ı�����Ϊlabels,����֮��Ľ�����Ƕ�ά�������������ȶ���һ���봦����������������ڶ�ÿһ�����������������룬����ı�������һ���������ֵ䣺

```python
def process_func(example):
    MAX_LENGTH = 512
    input_ids, labels = [], []
    prompt = tokenizer.encode("�û�:\n"+"������Ҫ���ݻʵ���ߵ�Ů��--��֡�", add_special_tokens=False)
    instruction_ = tokenizer.encode("\n".join([example["instruction"], example["input"]]).strip(), add_special_tokens=False,max_length=512)
    instruction = tokenizer.encode(prompt + instruction_)
    response = tokenizer.encode("CharacterGLM-6B:\n:" + example["output"], add_special_tokens=False)
    input_ids = instruction + response + [tokenizer.eos_token_id]
    labels = [tokenizer.pad_token_id] * len(instruction) + response + [tokenizer.eos_token_id]
    pad_len = MAX_LENGTH - len(input_ids)
    # print()
    input_ids += [tokenizer.pad_token_id] * pad_len
    labels += [tokenizer.pad_token_id] * pad_len
    labels = [(l if l != tokenizer.pad_token_id else -100) for l in labels]

    return {
        "input_ids": input_ids,
        "labels": labels
    }
```

������ʽ�������ݣ�Ҳ��������ģ�͵�ÿһ�����ݣ�����һ���ֵ䣬������input_ids��labels������ֵ�ԣ�����input_ids�������ı��ı��룬labels������ı��ı��롣

## ����tokenizer�Ͱ뾫��ģ��

ģ���԰澫����ʽ���أ�����Կ��Ƚ��£�������torch.bfloat��ʽ���أ������Զ����ģ��һ��Ҫָ��trust_remote_code����ΪTrue

```python
tokenizer=AutoTokenizer.from_pretrained('/root/autodl-tmp/THUCoAI/CharacterGLM-6B',use_fast=False,trust_remote_code=True)

model=AutoModelForCausalLM.from_pretrained('/root/autodl-tmp/THUCoAI/CharacterGLM-6B',trust_remote_code=True,torch_dtype=torch.half,device_map="auto")
```

## ����LoraConfig

LoraConfig������п������úܶ���������ֲ���չʾ���£�
task_type:ģ������
target����modules����Ҫѵ����ģ�Ͳ�����֣���Ҫ����attention���ֵĲ㣬��ͬ��ģ�Ͷ�Ӧ�Ĳ�����ֲ�ͬ�����Դ������飬Ҳ�����ַ�����Ҳ����������ʽ��
r:lora����
lora_alpha:Lora alpha
modules_to_save:ָ�����ǳ��˲��lora��ģ�飬������ģ�����������ָ��ѵ��

Lora������ʽlora_alpha/r�������LoraConfig�����ž���4����������ŵı��ʲ�û�иı�Lora�Ĳ�������С���������ڽ�����Ĳ�����ֵ���㲥�˷����������Ե����š�

```python
config=LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    target_modules=["query_key_value"],
    inference_mode=False,
    r=8,
    lora_alpha=32,
    lora_dropout=0.1
)
```

## �Զ���TraininArguments����

TrainingArguments������Դ��Ҳ������ÿ�������ľ������ã����õĲ������£�
output_dir:ģ�͵����·��
per_device_train_batch_size:batch_size
gradient_accumulation_steps:�ݶ��ۼӣ�����Դ�Ƚ�С�����԰�batch_size����Сһ�㣬�ݶ��ۻ�����һ��
logging_steps:���ٲ������һ��log
num_train_epochs:����˼��epoch
gradient_chechpointing:�ݶȼ�飬���һ��������ģ�;ͱ���ִ��
model.enable_input_require_grads()

```python
data_collator=DataCollatorForSeq2Seq(
    tokenizer,
    model=model,
    label_pad_token_id=-100,
    pad_to_multiple_of=None,
    padding=False
)
args=TrainingArguments(
    output_dir="./output/CharacterGLM",
    per_device_train_batch_size=4,
    gradient_accumulation_steps=2,
    logging_steps=10,
    num_train_epochs=3,
    gradient_checkpointing=True,
    save_steps=100,
    learning_rate=1e-4,
)
```

## ʹ��Trainerѵ��

��model�Ž�ȥ�����������õĲ����Ž�ȥ�����ݼ��Ž�ȥ����ʼѵ��

```python
trainer=Trainer(
    model=model,
    args=args,
    train_dataset=tokenized_id,
    data_collator=data_collator,
)
trainer.train()
```

## ģ������

```python
model = model.cuda()
ipt = tokenizer("�û���{}\n{}".format("������Ҫ���ݻʵ���ߵ�Ů��--��֡�����˭��", "").strip() + "characterGLM-6B:\n", return_tensors="pt").to(model.device)
tokenizer.decode(model.generate(**ipt, max_length=128, do_sample=True)[0], skip_special_tokens=True)
```

## ���¼���

ͨ��PEFT��΢����ģ�ͣ�������ʹ������ķ����������¼��أ�������

����Դmodel��tokenizer��
ʹ��PeftModel�ϲ�Դmodel��PEFT΢����Ĳ���

```python
from peft import Peftmodel
model=AutoModelForCausalLM.from_pretrained("/root/autodl-tmp/THUCoAI/CharacterGLM-6B",trust_remote_code=True,low_cpu_mem_usage=True)
tokenizer=AutoTokenizer.from_pretrained("root/autodl-tmp/THUCoAI/CharacterGLM-6B",use_fast=False,trust_remote_code=True)
p_model=PeftModel.from_pretrained(model,model_id="./output/CharatcerGLM/checkpoint-1000/")
ipt = tokenizer("�û���{}\n{}".format("������Ҫ���ݻʵ���ߵ�Ů��--��֡�����˭��", "").strip() + "characterGLM-6B:\n", return_tensors="pt").to(model.device)
tokenizer.decode(p_model.generate(**ipt,max_length=128,do_sample=True)[0],skip_special_tokens=True)
```
