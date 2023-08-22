---
layout: post
title: "chatGLM fintune"
description: ""
categories: [NLP]
tags: []
redirect_from:
  - /2023/07/11/
---

* Karmdown table of content
{:toc .toc}

# 前言

chatGLM-6b在32GB的tesla v100上即可进行部署，并且完成one-shot的信息抽取，也可以通过ptuning进行单卡fintune。

个人认为，大模型微调最大的优势在于，凡是可以转换为问答形式的任务，都可以在不改变模型结构的情况下，完成训练和预测。与以往标数据-搭模型-训练-预测过程相比，用户唯一需要做的，就是把标注数据转换为问答对，搭模型、训练、预测过程均不需要进行太多工作。

大模型在微调过程中，依然无法避免数据遗忘的问题，而且对于分布较少的类别，大模型很难预测到结果。个人观点，大模型适合在短时间内完成一个展示demo，如果需要提升效果，依然需要针对任务，使用该任务下最好的模型结构。

下面对本人通过大模型微调完成信息抽取任务的过程进行记录。本人的实验环境为V100\*2，训练过程使用p-tuning

# chatGLM 部署及one-shot实现信息抽取

参考博客：[官方github](https://github.com/THUDM/ChatGLM-6B)

## pull代码及模型


### pull 代码

~~~
git clone https://github.com/THUDM/ChatGLM-6B.git
~~~

### 安装环境

~~~
pip install -r /content/ChatGLM-6B/requirements.txt
~~~

### 下载模型


在ChatGLM-6B路径下下载模型的py和json文件：

~~~
git clone https://huggingface.co/THUDM/chatglm-6b
~~~


由于github传输大文件比较麻烦，上述命令下载checkpoint（.bin文件）往往会失败。所以建议使用清华云的链接下载。命令如下：

~~~
wget -P 'ChatGLM-6B/model' --no-check-certificate --content-disposition 'https://cloud.tsinghua.edu.cn/d/fb9f16d6dc8f482596c2/files/?p=%2Fpytorch_model-00001-of-00008.bin&dl=1'
wget -P 'ChatGLM-6B/model' --no-check-certificate --content-disposition 'https://cloud.tsinghua.edu.cn/d/fb9f16d6dc8f482596c2/files/?p=%2Fpytorch_model-00002-of-00008.bin&dl=1'
wget -P 'ChatGLM-6B/model' --no-check-certificate --content-disposition 'https://cloud.tsinghua.edu.cn/d/fb9f16d6dc8f482596c2/files/?p=%2Fpytorch_model-00003-of-00008.bin&dl=1'
wget -P 'ChatGLM-6B/model' --no-check-certificate --content-disposition 'https://cloud.tsinghua.edu.cn/d/fb9f16d6dc8f482596c2/files/?p=%2Fpytorch_model-00004-of-00008.bin&dl=1'
wget -P 'ChatGLM-6B/model' --no-check-certificate --content-disposition 'https://cloud.tsinghua.edu.cn/d/fb9f16d6dc8f482596c2/files/?p=%2Fpytorch_model-00005-of-00008.bin&dl=1'
wget -P 'ChatGLM-6B/model' --no-check-certificate --content-disposition 'https://cloud.tsinghua.edu.cn/d/fb9f16d6dc8f482596c2/files/?p=%2Fpytorch_model-00006-of-00008.bin&dl=1'
wget -P 'ChatGLM-6B/model' --no-check-certificate --content-disposition 'https://cloud.tsinghua.edu.cn/d/fb9f16d6dc8f482596c2/files/?p=%2Fpytorch_model-00007-of-00008.bin&dl=1'
wget -P 'ChatGLM-6B/model' --no-check-certificate --content-disposition 'https://cloud.tsinghua.edu.cn/d/fb9f16d6dc8f482596c2/files/?p=%2Fpytorch_model-00008-of-00008.bin&dl=1'
~~~

## 代码调用

这里以信息抽取任务为例，演示通过代码调用chatGLM的方式。


~~~python
#!/usr/bin/env python
# _*_coding:utf-8_*_
# Author   :    Junhui Yu

import re
import json
from transformers import AutoTokenizer, AutoModel

# 根据需求补充
schema = {
    'JD岗位要求': ['学历要求', '专业要求', '工作年限要求', '编程语言', '加分项']
}

IE_PATTERN = "{}\n\n提取上述句子中{}类型的实体，输出JSON格式，上述句子中不存在的信息用['该JD未要求']来表示，多个值之间用','分隔。"

ie_examples = {
    'JD岗位要求':
        {
            'sentence': '职位要求：1、硕士以上学历。2、计算机相关专业。3、3年以上工作经验。4、熟练掌握python或者c++语言。5、有自然语言处理获奖经历优先',
            'answers': {
                '学历要求': ['硕士'],
                '专业要求': ['计算机'],
                '工作年限要求': ['3年以上'],
                '编程语言': ['python', 'c++'],
                '加分项': ['自然语言处理获奖经历'],
            }
        }
}


def init_prompts():
    ie_prefix = [
        (
            "需要你协助完成信息抽取任务，当我给你一个JD职位要求时，帮我抽取出句子中三元组，并按照JSON的格式输出，上述句子中没有的信息用['该JD未要求']来表示，多个值之间用','分隔。",
            '请输入JD职位描述。'
        )
    ]
    properties_str = ', '.join(schema['JD岗位要求'])
    schema_str_list = f'“JD岗位要求”({properties_str})'
    sentence = ie_examples['JD岗位要求']['sentence']
    sentence_with_prompt = IE_PATTERN.format(sentence, schema_str_list)
    ie_prefix.append((
        f'{sentence_with_prompt}',
        f"{json.dumps(ie_examples['JD岗位要求']['answers'], ensure_ascii=False)}"
    ))

    return {'ie_prefix': ie_prefix}


def format_output(response: str):
    if '```json' in response:
        res = re.findall(r'```json(.*?)```', response)
        if len(res) and res[0]:
            response = res[0]
        response.replace('、', ',')
    try:
        return json.loads(response)
    except:
        return response


def inference(sentences: list, custom_settings: dict):
    for sentence in sentences:
        properties_str = ', '.join(schema['JD岗位要求'])
        schema_str_list = f'“JD岗位要求”({properties_str})'
        sentence_with_ie_prompt = IE_PATTERN.format(sentence, schema_str_list)
        result, _ = model.chat(tokenizer, sentence_with_ie_prompt, history=custom_settings['ie_prefix'])
        result = format_output(result)
        print(sentence)
        print(result)


if __name__ == '__main__':
    tokenizer = AutoTokenizer.from_pretrained("chatglm-6b", trust_remote_code=True)
    model = AutoModel.from_pretrained("chatglm-6b", trust_remote_code=True).half().cuda()

    sentences = [
        '职位要求：1、本科以上学历。2、电子信息或软件工程专业。3、1-3年工作经验。4、熟练掌握java或者c++语言。5、有相关项目经验优先',
    ]

    custom_settings = init_prompts()
    inference(sentences,
        custom_settings
    )
~~~

直接执行上述py文件即可。跑通之后可以尝试更改prompt和one-shot的实例，来简单尝试提升效果。也可以进行finetune。

# fintune 

参考博客：https://github.com/HarderThenHarder/transformers_tasks/blob/main/LLM/finetune/readme.md

从上述网址的github下载好代码之后，需要按照上一章节中的方式把模型也下载下来。

## 安装环境


由于 ChatGLM 需要的环境和该项目中其他实验中的环境有所不同，因此我们强烈建议您创建一个新的虚拟环境来执行该目录下的全部代码。

下面，我们将以 Anaconda 为例，展示如何快速搭建一个环境：

创建一个虚拟环境，您可以把 llm_env 修改为任意你想要新建的环境名称：

~~~
conda create -n llm_env python=3.8
~~~

激活新建虚拟环境并安装响应的依赖包：

~~~
conda activate llm_env
pip install -r requirements.txt
~~~

安装对应版本的 peft：
~~~
cd peft-chatglm
python setup.py install
~~~

## 数据集准备

在该实验中，我们将尝试使用 信息抽取 + 文本分类 任务的混合数据集喂给模型做 finetune，数据集在 data/mixed_train_dataset.jsonl。

每一条数据都分为 context 和 target 两部分：

> context 部分是接受用户的输入。
> target 部分用于指定模型的输出。

在 context 中又包括 2 个部分：

> Instruction：用于告知模型的具体指令，当需要一个模型同时解决多个任务时可以设定不同的 Instruction 来帮助模型判别当前应当做什么任务。
> Input：当前用户的输入。

信息抽取数据示例
Instruction 部分告诉模型现在需要做「阅读理解」任务，Input 部分告知模型要抽取的句子以及输出的格式。

~~~
{
    "context": "Instruction: 你现在是一个很厉害的阅读理解器，严格按照人类指令进行回答。\nInput: 找到句子中的三元组信息并输出成json给我:\n\n九玄珠是在纵横中文网连载的一部小说，作者是龙马。\nAnswer: ", 
    "target": "```json\n[{\"predicate\": \"连载网站\", \"object_type\": \"网站\", \"subject_type\": \"网络小说\", \"object\": \"纵横中文网\", \"subject\": \"九玄珠\"}, {\"predicate\": \"作者\", \"object_type\": \"人物\", \"subject_type\": \"图书作品\", \"object\": \"龙马\", \"subject\": \"九玄珠\"}]\n```"
}
~~~

文本分类数据示例
Instruction 部分告诉模型现在需要做「阅读理解」任务，Input 部分告知模型要抽取的句子以及输出的格式。

~~~
{
    "context": "Instruction: 你现在是一个很厉害的阅读理解器，严格按照人类指令进行回答。\nInput: 下面句子可能是一条关于什么的评论，用列表形式回答：\n\n很不错，很新鲜，快递小哥服务很好，水果也挺甜挺脆的\nAnswer: ", 
    "target": "[\"水果\"]"
}
~~~



## 模型训练

### 单卡训练

实验中支持使用 LoRA Finetune 和 P-Tuning 两种微调方式。


运行 train.sh 文件，根据自己 GPU 的显存调节 batch_size, max_source_seq_len, max_target_seq_len 参数：

~~~sh
# LoRA Finetune
python train.py \
    --train_path data/mixed_train_dataset.jsonl \
    --dev_path data/mixed_dev_dataset.jsonl \
    --use_lora True \
    --lora_rank 8 \
    --batch_size 1 \
    --num_train_epochs 2 \
    --save_freq 1000 \
    --learning_rate 3e-5 \
    --logging_steps 100 \
    --max_source_seq_len 400 \
    --max_target_seq_len 300 \
    --save_dir checkpoints/finetune \
    --img_log_dir "log/fintune_log" \
    --img_log_name "ChatGLM Fine-Tune" \
    --device cuda:0


# P-Tuning
python train.py \
    --train_path data/mixed_train_dataset.jsonl \
    --dev_path data/mixed_dev_dataset.jsonl \
    --use_ptuning True \
    --pre_seq_len 128 \
    --batch_size 1 \
    --num_train_epochs 2 \
    --save_freq 200 \
    --learning_rate 2e-4 \
    --logging_steps 100 \
    --max_source_seq_len 400 \
    --max_target_seq_len 300 \
    --save_dir checkpoints/ptuning \
    --img_log_dir "log/fintune_log" \
    --img_log_name "ChatGLM P-Tuning" \
    --device cuda:0
~~~


成功运行程序后，会看到如下界面：

~~~
global step 900 ( 49.89% ) , epoch: 1, loss: 0.78065, speed: 1.25 step/s, ETA: 00:12:05
global step 1000 ( 55.43% ) , epoch: 2, loss: 0.71768, speed: 1.25 step/s, ETA: 00:10:44
Model has saved at checkpoints/model_1000.
Evaluation Loss: 0.17297
Min eval loss has been updated: 0.26805 --> 0.17297
Best model has saved at checkpoints/model_best.
global step 1100 ( 60.98% ) , epoch: 2, loss: 0.66633, speed: 1.24 step/s, ETA: 00:09:26
global step 1200 ( 66.52% ) , epoch: 2, loss: 0.62207, speed: 1.24 step/s, ETA: 00:08:06
~~~

在 log/finetune_log 下会看到训练 loss 的曲线图

### 多卡训练

运行 train_multi_gpu.sh 文件，通过 CUDA_VISIBLE_DEVICES 指定可用显卡，num_processes 指定使用显卡数：

~~~sh
# LoRA Finetune
CUDA_VISIBLE_DEVICES=0,1 accelerate launch --multi_gpu --mixed_precision=fp16 --num_processes=2 train_multi_gpu.py \
    --train_path data/mixed_train_dataset.jsonl \
    --dev_path data/mixed_dev_dataset.jsonl \
    --use_lora True \
    --lora_rank 8 \
    --batch_size 1 \
    --num_train_epochs 2 \
    --save_freq 500 \
    --learning_rate 3e-5 \
    --logging_steps 100 \
    --max_source_seq_len 400 \
    --max_target_seq_len 300 \
    --save_dir checkpoints_parrallel/finetune \
    --img_log_dir "log/fintune_log" \
    --img_log_name "ChatGLM Fine-Tune(parallel)"


# P-Tuning
CUDA_VISIBLE_DEVICES=0,1 accelerate launch --multi_gpu --mixed_precision=fp16 --num_processes=2 train_multi_gpu.py \
    --train_path data/mixed_train_dataset.jsonl \
    --dev_path data/mixed_dev_dataset.jsonl \
    --use_ptuning True \
    --pre_seq_len 128 \
    --batch_size 1 \
    --num_train_epochs 2 \
    --save_freq 500 \
    --learning_rate 2e-4 \
    --logging_steps 100 \
    --max_source_seq_len 400 \
    --max_target_seq_len 300 \
    --save_dir checkpoints_parrallel/ptuning \
    --img_log_dir "log/fintune_log" \
    --img_log_name "ChatGLM P-Tuning(parallel)"
~~~

相同数据集下，单卡使用时间：
~~~
Used 00:27:18.
~~~
多卡（2并行）使用时间：
~~~
Used 00:13:05.
~~~



## 模型预测

修改训练模型的存放路径，运行 python inference.py 以测试训练好模型的效果：

~~~py
# !/usr/bin/env python3
"""
==== No Bugs in code, just some Random Unexpected FEATURES ====
┌─────────────────────────────────────────────────────────────┐
│┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐│
││Esc│!1 │@2 │#3 │$4 │%5 │^6 │&7 │*8 │(9 │)0 │_- │+= │|\ │`~ ││
│├───┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴───┤│
││ Tab │ Q │ W │ E │ R │ T │ Y │ U │ I │ O │ P │{[ │}] │ BS  ││
│├─────┴┬──┴┬──┴┬──┴┬──┴┬──┴┬──┴┬──┴┬──┴┬──┴┬──┴┬──┴┬──┴─────┤│
││ Ctrl │ A │ S │ D │ F │ G │ H │ J │ K │ L │: ;│" '│ Enter  ││
│├──────┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴────┬───┤│
││ Shift  │ Z │ X │ C │ V │ B │ N │ M │< ,│> .│? /│Shift │Fn ││
│└─────┬──┴┬──┴──┬┴───┴───┴───┴───┴───┴──┬┴───┴┬──┴┬─────┴───┘│
│      │Fn │ Alt │         Space         │ Alt │Win│   HHKB   │
│      └───┴─────┴───────────────────────┴─────┴───┘          │
└─────────────────────────────────────────────────────────────┘

inference 训练好的模型。

Author: pankeyu
Date: 2023/03/17
"""
import time
import torch

from transformers import AutoTokenizer, AutoModel
torch.set_default_tensor_type(torch.cuda.HalfTensor)


def inference(
        model,
        tokenizer,
        instuction: str,
        sentence: str
    ):
    """
    模型 inference 函数。

    Args:
        instuction (str): _description_
        sentence (str): _description_

    Returns:
        _type_: _description_
    """
    with torch.no_grad():
        input_text = f"Instruction: {instuction}\n"
        if sentence:
            input_text += f"Input: {sentence}\n"
        input_text += f"Answer: "
        batch = tokenizer(input_text, return_tensors="pt")
        out = model.generate(
            input_ids=batch["input_ids"].to(device),
            max_new_tokens=max_new_tokens,
            temperature=0
        )
        out_text = tokenizer.decode(out[0])
        answer = out_text.split('Answer: ')[-1]
        return answer


if __name__ == '__main__':
    from rich import print

    device = 'cuda:0'
    max_new_tokens = 300
    model_path = "checkpoints/model_1000"

    tokenizer = AutoTokenizer.from_pretrained(
        model_path, 
        trust_remote_code=True
    )

    model = AutoModel.from_pretrained(
        model_path,
        trust_remote_code=True
    ).half().to(device)

    samples = [
        {
            'instruction': "给你一个句子，同时给你一个句子主语和关系列表，你需要找出句子中主语包含的所有关系以及关系值，并输出为SPO列表。",
            "input": "给定主语：绵阳电影院\n给定句子：影院简介绵阳电影院位于绵阳市区最繁华的黄金地段，紧邻好又多，东面梅西百货，南接大观园，西面绵阳文化广场，北攘肯德基麦当劳，交通便利，是绵阳商业娱乐文化中心。绵阳电影院占地1000多平米，建筑面积3000多平米。影院采用了国际标准设计，即高空间、大坡度、超亮屏幕，视觉无遮挡。配备有SR.D、DTS顶级数码立体声还音系统，音响效果极佳。再配以高级航空座椅，日本三菱空调，以及优质的人文服务，堪称川西北一流影院。绵阳电影院拥有数字大厅一个，豪华数字厅两个（2、4厅），纯数字电影两个（3、5厅），影厅内安装有世界顶级的英国杜比CP650(EX)数字处理器、美国JBL音响、德国ISCO一体化镜头、美国QSC数字功放（DCA）、5.1声道杜比数码立体声系统！精彩电影、适中价位、舒适享受，尽在绵阳电影院。同时兼营小卖，水吧，服装等，年收入500多万。其中票房收入位于四川省单厅（大厅）之首。绵阳电影院还被四川省人民政府授予“文明电影院”称号。绵阳电影院隶属于绵阳市川涪影业股份有限责任公司，是四川省太平洋电影院线旗下的旗舰影院，绵阳电影院拥有中影进口大片首轮放映权。绵阳电影院成立50多年来，凭借优质服务、一流的硬件设施、优秀的地理位置赢得了广大绵阳人民的厚爱。\n给定关系列表：['所属国家', '建造者', '面积', '设计者', '高度']",
        },
        {
            'instruction': "给你一个句子，同时给你一个句子主语和关系列表，你需要找出句子中主语包含的所有关系以及关系值，并输出为SPO列表。",
            "input": "句子主语：我是一片云\n输入句子:《我是一片云》是根据琼瑶同名小说改编，由辜朗辉、赵俊宏执导，张晓敏、王伟平、孙启新、秦怡、严丽秋等主演的电视剧。该剧共5集，于1985年播出。\n输入关系:['主演', '演员', '作品类型', '首发时间', '导演', '播出平台', '首播平台', '集数']",
        }
    ]

    start = time.time()
    for i, sample in enumerate(samples):
        res = inference(
            model,
            tokenizer,
            sample['instruction'],
            sample['input']
        )
        print(f'res {i}: ')
        print(res)
    print(f'Used {round(time.time() - start, 2)}s.')
~~~

