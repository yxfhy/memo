# GPT-OSSモデルの4bit量子化読込方法

import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = "openai/gpt-oss-20b"  # smaller GPT-OSS model (21B params, 3.6B active)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    load_in_4bit=True,              # load weights in 4-bit quantized form
    device_map="auto",              # automatically allocate to GPU/CPU
    torch_dtype=torch.float16,      # use FP16 computations for speed & memory
    max_memory={0: "10GiB", "cpu": "30GiB"}  # limit GPU to ~10GiB, rest to CPU
)
tokenizer = AutoTokenizer.from_pretrained(model_name, use_fast=False)

作成日時: 2025-08-06 08:50:31