# FineTuneLib
A library for seamless fintuning of LLMs

## REQUIREMENTS
 ```bash
  git clone https://github.com/OpenAccess-AI-Collective/axolotl
  cd axolotl

  pip3 install -e .
  pip3 install protobuf==3.20.3
  pip3 install -U --ignore-installed requests Pillow psutil scipy
  ```

## FINETUNE THE MODEL

Run
```bash
accelerate launch scripts/finetune.py your_config.yml
```
### Config:

See [examples](examples) for quick start. It is recommended to duplicate and modify to your needs. The most important options are:

- model
  ```yaml
  base_model: ./llama-7b-hf # local or huggingface repo
  ```
  Note: The code will load the right architecture.

- dataset
  ```yaml
  sequence_len: 2048 # max token length for prompt

  # huggingface repo
  datasets:
    - path: vicgalle/alpaca-gpt4
      type: alpaca # format from earlier

  # huggingface repo with specific configuration/subset
  datasets:
    - path: EleutherAI/pile
      name: enron_emails
      type: completion # format from earlier

  # huggingface repo with multiple named configurations/subsets
  datasets:
    - path: bigcode/commitpackft
      name:
        - ruby
        - python
        - typescript
      type: ... # unimplemented custom format

  # local
  datasets:
    - path: data.jsonl # or json
      ds_type: json # see other options below
      type: alpaca
  ```

- quantization
  ```yaml
  load_in_4bit: true
  load_in_8bit: true
  bf16: true # require >=ampere
  fp16: true
  tf32: true # require >=ampere
  bfloat16: true # require >=ampere, use instead of bf16 when you don't want AMP (automatic mixed precision)
  float16: true # use instead of fp16 when you don't want AMP
  ```
  Note: Repo does not do 4-bit quantization.

- lora
  ```yaml
  adapter: lora # qlora or leave blank for full finetune
  lora_r: 8
  lora_alpha: 16
  lora_dropout: 0.05
  lora_target_modules:
    - q_proj
    - v_proj
  ```

## Inference

Pass the appropriate flag to the train command:

- Pretrained LORA:
  ```bash
  --inference --lora_model_dir="./lora-output-dir"
  ```
- Full weights finetune:
  ```bash
  --inference --base_model="./completed-model"
  ```
- Full weights finetune w/ a prompt from a text file:
  ```bash
  cat /tmp/prompt.txt | python scripts/finetune.py configs/your_config.yml \
    --base_model="./completed-model" --inference --prompter=None --load_in_8bit=True
  ```

## Merge LORA to base

Add below flag to train command above

```bash
--merge_lora --lora_model_dir="./completed-model" --load_in_8bit=False --load_in_4bit=False
```

If you run out of CUDA memory, you can try to merge in system RAM with

```bash
CUDA_VISIBLE_DEVICES="" python3 scripts/finetune.py ...
```
