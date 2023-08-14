# ⛰️Valley: Video Assistant with Large Language model Enhanced abilitY
Understanding Complex Videos Relying on Large Language and Vision Models

[[Project Page](https://valley-vl.github.io/)] [[Paper](https://arxiv.org/pdf/2306.07207.pdf)]~[[demo]()]~

The online demo is no longer available, because we released the code for offline demo deployment

**Video Assistant with Large Language model Enhanced abilitY** <br>
[Ruipu Luo*](https://github.com/RupertLuo), [Ziwang Zhao*](), [Min Yang*](https://github.com/feymanpriv) (*Equal Contribution)

<p align="center">
    <img src="valley/logo/lama_with_valley.jpeg" width="100%"><br>
    Generated by <a href="https://stablecog.com/">stablecog</a> via "A cute llama with valley"
</p>

[![Code License](https://img.shields.io/badge/Code%20License-Apache_2.0-green.svg)](https://github.com/tatsu-lab/stanford_alpaca/blob/main/LICENSE)
[![Data License](https://img.shields.io/badge/Data%20License-CC%20By%20NC%204.0-red.svg)](https://github.com/tatsu-lab/stanford_alpaca/blob/main/DATA_LICENSE)
**Usage and License Notices**: The data, code and checkpoint is intended and licensed for research use only. They are also restricted to uses that follow the license agreement of LLaMA, Vicuna and GPT-4. The dataset is CC BY NC 4.0 (allowing only non-commercial use) and models trained using the dataset should not be used outside of research purposes.

## Release
- [8/10] 🔥 Realeased pretrain stage weight of 13b and 7b ,[Valley2-7b-pretrain](https://huggingface.co/luoruipu1/Valley2-7b-pretrain/), [valley-13b-pretrain](https://huggingface.co/luoruipu1/valley-13b-pretrain)
- [8/8] 🔥 We released the self-collected and expanded instruction fine-tuning dataset ([Valley-Instruct-73k](https://huggingface.co/datasets/luoruipu1/Valley-Instruct-73k)).
- [8/7]  🔥 We released [Valley2-7b](https://huggingface.co/luoruipu1/Valley2-7b), It replaces Vicuna with Llama 2.
- [7/23] 🫧 We modified the our training code to make it easier to train valley and also support the training of lora.
- [7/5]  🫧 Release training code for valley, and upload our pretraining data 
- [6/21] 🫧 upload offline demo code.
- [6/14] 🫧 build a share link ~[[demo]()]~.
- [6/13] 🫧 We uploaded model weight of [Valley-13b-v1-delta](https://huggingface.co/luoruipu1/valley-13b-v1-delta).
- [6/12] 🫧 We released Valley: Video Assistant with Large Language model Enhanced abilitY.  Checkout the [paper](https://arxiv.org/pdf/2306.07207.pdf).

## Todo
- ~~Release inference code~~
- ~~Upload weight of **Valley-v1** and build a share link demo~~
- ~~Upload offline demo code~~
- ~~Release 703k pretraining data and 73k instruction tuning data~~ 
- ~~Upload pretrain and tuning code~~
- ~~Upload weight of Valley2-7B~~ and Valley-v3

## Install
1. Clone this repository and navigate to Valley folder
```
git clone https://github.com/RupertLuo/Valley.git
cd Valley
```
2. Install Package
```
conda create -n valley python=3.10 -y
conda activate valley
pip install --upgrade pip 
pip install -e .
```
## ValleyWeight
We release [Valley]() delta weights weights to comply with the LLaMA model license. You can apply this delta weights to original LLaMA model weight through the instructions blew:

1. Get the original LLaMA weights in the huggingface format by following the instructions structions [here](https://huggingface.co/docs/transformers/main/model_doc/llama).
2. Use the following scripts to get Valley weights by applying our delta ([13b-v1](https://huggingface.co/luoruipu1/valley-13b-v1-delta)).
### Valley 13b v1
```bash
python3 valley/model/apply_delta.py \
    --base /path/to/llama-13b \
    --target /output/path/to/Valley-13B-v1 \
    --delta /path/to/valley-13b-v1-delta
```
## Web UI
<p align="center">
    <img src="valley/logo/demo.GIF" width="100%"><br>
</p>

The framework of this webUI comes from [LLaVA](https://github.com/haotian-liu/LLaVA) and [FastChat](https://github.com/lm-sys/FastChat), we modified a part of the code to make this demo support the input of video and images.
#### launch a controller
```bsah
python valley/serve/controller.py
```
#### launch a model worker
```bsah
python valley/serve/model_worker.py --model-path /path/to/valley-13b-v1
```
Ps: At present, only single card mode is supported to load the model, and at least 30G of video memory is required, so the graphics card needs at least one Tesla V100.
#### launch a gradio demo 
```bash
python valley/serve/gradio_web_server_video.py --share
```


## Inference Valley in Command Line
We now update inference code which is more convient, and supports input in the form of openai api.

Inference CLI
```
python3 inference/run_valley.py --model-name [PATH TO VALLEY WEIGHT] --video_file [PATH TO VIDEO] --quary [YOUR QUERY ON THE VIDEO]
```

Inference in code
```python
from transformers import AutoTokenizer
from valley.model.valley import ValleyLlamaForCausalLM
def init_vision_token(model,tokenizer):
    vision_config = model.get_model().vision_tower.config
    vision_config.im_start_token, vision_config.im_end_token = tokenizer.convert_tokens_to_ids([DEFAULT_IM_START_TOKEN, DEFAULT_IM_END_TOKEN])
    vision_config.vi_start_token, vision_config.vi_end_token = tokenizer.convert_tokens_to_ids([DEFAULT_VI_START_TOKEN, DEFAULT_VI_END_TOKEN])
    vision_config.vi_frame_token = tokenizer.convert_tokens_to_ids(DEFAULT_VIDEO_FRAME_TOKEN)
    vision_config.im_patch_token = tokenizer.convert_tokens_to_ids([DEFAULT_IMAGE_PATCH_TOKEN])[0]

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
# input the query
query = "Describe the video concisely."
# input the systemprompt
system_prompt = "You are Valley, a large language and vision assistant trained by ByteDance. You are able to understand the visual content or video that the user provides, and assist the user with a variety of tasks using natural language. Follow the instructions carefully and explain your answers in detail."

model_path = THE MODEL PATH
model = ValleyLlamaForCausalLM.from_pretrained(model_path, torch_dtype=torch.float16)
tokenizer = AutoTokenizer.from_pretrained(model_path)
init_vision_token(model,tokenizer)
model = model.to(device)
model.eval()

# we support openai format input
message = [ {"role":'system','content':system_prompt},
            {"role":"user", "content": 'Hi!'},
            {"role":"assistent", "content": 'Hi there! How can I help you today?'},
            {"role":"user", "content": query}]

gen_kwargs = dict(
    do_sample=True,
    temperature=0.2,
    max_new_tokens=1024,
)
response = model.completion(tokenizer, args.video_file, message, gen_kwargs, device)
```


## Train Valley Step By Step

Inspired by LLAVA, we adopt a two-stage training method. The pre-training stage uses the [Valley-webvid2M-Pretrain-703K](https://huggingface.co/datasets/luoruipu1/Valley-webvid2M-Pretrain-703K) and [LLaVA-CC3M-Pretrain-595K](https://huggingface.co/datasets/liuhaotian/LLaVA-Pretrain).  And fine-tune stage uses [LLaVA-instruct-150K](https://huggingface.co/datasets/liuhaotian/LLaVA-Instruct-150K) ,  [VideoChat-instruct-11K](https://github.com/OpenGVLab/InternVideo/tree/main/Data/instruction_data)  and [Valley-Instruct-73k](https://huggingface.co/datasets/luoruipu1/Valley-Instruct-73k)

We modified our code for training valley and managed the model hyperparameters with yaml files. Run the following two scripts to perform valley training.

### Pretrain
The llm backbone that currently supports pre-training is Llama(7b,13b), vicuna(7b,13b), stable-vicuna(13b), Llama2(chat-7b, chat-13b). You need to download these open source language model weights yourself and convert them to the huggingface format. 
```shell
bash valley/train/train.sh valley/configs/experiment/valley_stage1.yaml
```

#### Finetune

```shell
bash valley/train/train.sh valley/configs/experiment/valley_stage2.yaml
```



## Acknowledgement

- [LLaVA](https://github.com/haotian-liu/LLaVA) & [MOSS](https://github.com/OpenLMLab/MOSS): Thanks to these two repositories for providing high-quality code, our code is based on them.
## Citation
If the project is helpful to your research, please consider citing our paper as follows

```bibtex
@misc{luo2023valley,
      title={Valley: Video Assistant with Large Language model Enhanced abilitY}, 
      author={Ruipu Luo and Ziwang Zhao and Min Yang and Junwei Dong and Minghui Qiu and Pengcheng Lu and Tao Wang and Zhongyu Wei},
      year={2023},
      eprint={2306.07207},
      archivePrefix={arXiv},
      primaryClass={cs.CV}
}
```
