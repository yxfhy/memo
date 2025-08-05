# GPT-OSS-7Bを11GB VRAMで動かす手順

Setting Up GPT-OSS-7B Inference with Hugging Face Transformers

1. Environment and Compatibility Verification

Before installing anything, ensure your system meets all requirements and that the software versions are compatible:

GPU & Memory: You have a single GPU with ~11 GB VRAM. A 7B parameter model typically requires ~14–15 GB in half-precision (FP16) memory, which exceeds 11 GB. This means we must use quantization (8-bit or 4-bit) to reduce memory usage so the model fits in VRAM. OpenAI’s GPT-OSS models are already released in a 4-bit format (MXFP4) – for example, the 20B model (21B params) runs within ~16 GB memory thanks to 4-bit weights. A smaller ~7B model in 4-bit should comfortably fit in 11 GB VRAM, and even the 20B can potentially run with CPU offloading.

CUDA and Drivers: CUDA 11.8 and cuDNN 9.0 are installed on your system. Verify that your NVIDIA driver is new enough for CUDA 11.8 (CUDA 11.8 typically requires driver version 520.xx or higher). If the driver is too old, you may encounter warnings or disabled GPU features. For example, bitsandbytes (the library used for 8-bit/4-bit inference) will warn if the driver is outdated and may fall back to CPU mode. Update the GPU driver if needed before proceeding.

Python Version: You are using Python 3.12. Not all deep learning libraries supported 3.12 initially, so you need recent versions. Use a PyTorch release that supports Python 3.12 (PyTorch added official 3.12 wheel support around early 2024). In practice, PyTorch 2.1+ or 2.2+ is recommended. We will install PyTorch with CUDA 11.8 to match your setup. Ensure pip, setuptools, and wheel are up to date in your environment, as Python 3.12 removed distutils which older build scripts relied on.

Hugging Face Libraries: We'll use the latest Hugging Face Transformers and Accelerate libraries. These support big model inference with device placement (device_map="auto") and integrate with bitsandbytes for quantization. We’ll also use bitsandbytes for 4-bit/8-bit loading, and the PEFT library for LoRA later. All of these now have Python 3.12 support, but using the latest versions is key (older versions might not have wheels for 3.12 or had bugs that are fixed in newer releases).


2. Installing Required Packages

With compatibility checked, proceed to install the necessary packages. It’s best to do this in a new virtual environment (e.g. via venv or Conda):

1. PyTorch (CUDA 11.8 build): Install PyTorch with CUDA 11.8 support. You can use the official pip wheel as follows:

pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

This will fetch a PyTorch build compiled for CUDA 11.8. (If this fails, ensure that a wheel exists for your Python version – PyTorch 2.1+ should support Py3.12. As of writing, PyTorch 2.5/2.6 are available and support CUDA 11.8 and Python 3.12.)


2. Transformers and Accelerate: Install Hugging Face Transformers and Accelerate. These libraries will orchestrate model loading and device offloading. For example:

pip install -U transformers accelerate

Using -U (upgrade) ensures you get the latest versions. The latest Accelerate is recommended because it contains fixes for 8-bit/4-bit device mapping and fewer conflicts with bitsandbytes (the HF team even recommended a specific dev commit for Python 3.12 in earlier guides).


3. BitsandBytes (8-bit/4-bit quantization): Install the bitsandbytes library, which enables 8-bit and 4-bit weight quantization in Transformers. For example:

pip install -U bitsandbytes

> Note: On Linux, bitsandbytes provides precompiled CUDA binaries. As of bitsandbytes 0.45+ it supports CUDA 11.8+; version 0.46 even dropped support for older CUDA toolkits. Ensure you get bitsandbytes >=0.45. If pip installs an older version (e.g. 0.42) by default, explicitly install a newer version (pip install bitsandbytes==0.45.2 or latest). You can verify a correct install by running python -m bitsandbytes – it should report that GPU acceleration is available, not "compiled without GPU support."




4. Hugging Face Hub (optional): Install huggingface_hub if not already included. This is often pulled in with Transformers, but you can ensure it's there to enable model downloading:

pip install -U huggingface-hub


5. Tokenizer Dependencies: The GPT-OSS models use the same tokenizer as OpenAI GPT-4 models (the cl100k_base tokenizer). Transformers should handle this automatically via the tiktoken library. If you encounter errors loading the tokenizer, install OpenAI’s tiktoken: pip install tiktoken. (In most cases, AutoTokenizer will manage this, possibly requiring trust_remote_code=True on load if the tokenizer is implemented in the model repo.)


6. PEFT (for LoRA): Install the PEFT library to prepare for LoRA fine-tuning later:

pip install -U peft

This will allow applying LoRA adapters to the model. The version should be 0.4+ or newer (0.8.x as of 2025).


7. (Optional) Other Tools: If you plan on experimenting with training or memory optimization, you might also install 🤗 Datasets for loading data, and evaluate for evaluation metrics (these are optional for just inference). For speed-ups, you could install xformers or NVIDIA’s flash-attn for faster attention, but these require custom builds and are optional. The core setup above is sufficient for running inference.



After installing, verify the versions: e.g., run python -c "import torch, transformers, bitsandbytes, accelerate, peft; print(torch.__version__, transformers.__version__, bitsandbytes.__version__, accelerate.__version__, peft.__version__)". Ensure torch reports a CUDA build (+cu118 for example) and bitsandbytes does not throw errors on import.

3. Downloading and Quantized Loading of the GPT-OSS-7B Model

With the environment ready, you can now download and load the GPT-OSS model. We will use Hugging Face’s transformers API to download the weights from the Hugging Face Hub and load them directly into a quantized format to save memory. (In this guide, we refer to “gpt-oss-7b” as the smaller GPT-OSS model; since OpenAI’s open models are 20B and 120B parameter MoEs, you might use the 20B variant and treat it as comparable to a dense 7B model due to 3.6B active parameters. The steps are the same for any 7B-ish model on Hugging Face.)

Steps to load the model in 8-bit or 4-bit:

Choose quantization: For 11GB VRAM, 4-bit is recommended to leave headroom (you can also try 8-bit if memory allows). We’ll demonstrate 4-bit (QLoRA style) loading. This uses bitsandbytes under the hood to load weights in NVidia’s NF4 4-bit format (which preserves accuracy well). If you prefer 8-bit, set load_in_8bit=True instead for slightly higher precision at the cost of ~2x memory vs 4-bit.

Auto device placement: We use device_map="auto" so that 🤗 Accelerate will automatically place as much of the model on the GPU as fits, and spill the rest to CPU RAM if necessary. This is crucial for large models. With 4-bit quantization, the entire 7B model should fit in 11GB, but if you use the 20B model, device_map="auto" will offload some layers to CPU since 16GB is recommended for 20B. We can also explicitly limit the GPU memory used by setting a max_memory mapping.

Load with Transformers: Use the AutoModelForCausalLM.from_pretrained method with the appropriate flags. For example:

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

This code will download the GPT-OSS-20B model from Hugging Face (approx. ~8GB of 4-bit weight files) and load it. Key points:

We pass load_in_4bit=True and device_map="auto" – this is the simple one-line way to get a quantized model on limited VRAM. Under the hood, this uses bitsandbytes to load the model in 4-bit NF4 format. By default it will use one quantization constant per weight row (smooth quantization) and a FP16 compute dtype for matrix multiplications (we set torch_dtype=torch.float16 explicitly to ensure FP32 isn’t used anywhere unnecessary).

We provided a max_memory dict to be safe: here we constrain the GPU (device 0) to use at most ~10 GiB for model weights, and allow up to 30 GiB of host RAM for offloading. Adjust these if needed (e.g. if you have 64GB system RAM, you can set a higher CPU limit). This ensures Accelerate won’t try to fill the entire 11GB (which could risk an OOM). With 4-bit quantization, the 7B model will likely use well under 10 GB, but the 20B might offload a few layers to stay within 10 GB.

We use AutoTokenizer to load the tokenizer. For GPT-OSS, the fast tokenizer might not be supported yet (hence use_fast=False to fall back to Python tokenizer if needed). The tokenizer will handle the special tokenization (it’s GPT-4’s tokenizer with 100k vocab).

Important: The GPT-OSS models were trained with a special Harmony chat format (with tokens like user etc.) and might behave oddly with plain prompts. For basic testing we can ignore this format, but keep in mind that for best results you should format prompts using the official template. (The Transformers pipeline can do this for you, as we’ll note below.)


> Note: If your from_pretrained call fails due to missing modules or undefined architecture, you may need to upgrade transformers or set trust_remote_code=True. The GPT-OSS model should be supported by latest transformers (the architecture is a MoE transformer similar to SwitchTransformers/NLLB-MoE, integrated in Transformers), so trust_remote_code is typically not required. But if you see errors about GPTOSSBlock or such not found, use trust_remote_code=True to allow loading the model’s custom code.




4. Running a Test Prompt

After loading, put the model in evaluation mode (model.eval()) and run a simple inference to verify everything is working.

We can test the model in two ways: using the Transformers pipeline API (with conversation format), or by manually calling model.generate on the model. We’ll demonstrate both:

A. Using pipeline (with chat format): The Transformers pipeline can simplify chat-based generation. Since GPT-OSS expects a chat format, we can leverage the pipeline to format the input. For example:

from transformers import pipeline

pipe = pipeline(
    "text-generation",
    model=model,       # use our loaded model & tokenizer
    tokenizer=tokenizer,
    device_map="auto",
    torch_dtype=torch.float16
)
# Compose a chat-style prompt (role-based)
messages = [
    {"role": "user", "content": "Explain quantum mechanics in simple terms."}
]
result = pipe(messages, max_new_tokens=100)
print(result[0]["generated_text"][-1])

Here, we pass a list of message dicts (with roles "user"/"assistant") as input. The pipeline will automatically apply the OpenAI Harmony format expected by GPT-OSS. We request 100 new tokens. The output from pipe is a list; we print the generated text from the first result.

Expected behavior: The model should produce a coherent explanation of quantum mechanics. If everything is set up correctly, you should see a reasonable answer. (If the output seems cut off or empty, ensure you’re printing the right key – for conversational pipelines, the generated text is nested under ["generated_text"][-1] as shown above.)

B. Using model.generate (manual prompt): Alternatively, you can feed a prompt as a single string. Keep in mind the model was tuned for a chat format, but it will still generate text for a direct prompt. For example:

prompt = "Q: What is the capital of France?\nA:"
inputs = tokenizer(prompt, return_tensors="pt").to(0)  # move inputs to GPU 0
outputs = model.generate(**inputs, max_new_tokens=50)
answer = tokenizer.decode(outputs[0], skip_special_tokens=True)
print(answer)

This will tokenize the prompt and generate an answer. We included a Q/A format in the prompt to encourage a direct answer. The decode step will convert token IDs back to text, skipping any special tokens. You should see the model respond with "Paris." or a short explanation.


In either case, verify that generation works without running out of memory. With 4-bit quantization and one 7B model on 11GB, you should have some VRAM to spare – monitor GPU memory during generation. The model’s forward pass should fit in memory since we’re using half-precision compute and quantized weights. If you get a CUDA out-of-memory error during model.generate, try reducing max_new_tokens (to generate a smaller output) or ensure no other processes are using GPU memory. Also, calling torch.cuda.empty_cache() before allocation can help if fragmentation from earlier steps is an issue.

5. Memory Optimization for Low-VRAM Scenarios

To ensure smooth inference on an 11GB card, we already leveraged the main tricks: 4-bit quantization and device_map offloading. Here are additional tips to optimize memory usage:

Use 4-bit (QLoRA) or 8-bit loading: We chose 4-bit (load_in_4bit=True) which drastically reduces model size in memory (roughly 1/4 of FP16). If you encounter any issues with 4-bit (compatibility or accuracy), 8-bit (load_in_8bit=True) is an alternative that still halves memory use vs FP16. Both modes use the bitsandbytes library under the hood. According to Hugging Face, any model architecture supported by Accelerate can be quantized in 4-bit or 8-bit this way.

CPU Offloading: With device_map="auto", any layers that don’t fit on the GPU will automatically live on CPU RAM. In our example we constrained GPU to 10 GiB, so some final layers might be on CPU. This makes inference slower (data transfers each time those layers are used), but prevents OOM. You can adjust the max_memory limits if you want to force more or less onto the GPU. As a rule of thumb, keep some VRAM free for safety (e.g. don’t fill 100% of GPU memory with model weights). The Unsloth team notes that GPT-OSS-20B in 4-bit runs in about 14 GB of memory total and can work with even less, at the cost of speed – so 11 GB VRAM + some CPU RAM is workable for that model, just expect token generation to be a bit slower due to offloading.

Enable half precision for activations: We set torch_dtype=torch.float16 when loading the model to ensure model layers use FP16 for computations. This saves memory on activations and speeds up GPU compute. (By default, Transformers might keep certain layers in float32; our setting forces them to FP16 where safe.) If your GPU supports bfloat16, you could use torch_dtype=torch.bfloat16 as an alternative with potentially better stability (bfloat16 doesn’t overflow as easily as FP16).

Disable grad tracking: Since we are only doing inference, wrap generation code in torch.no_grad() or use model.eval() (which we did). This prevents allocation of grad tensors. The Accelerate offload mechanism runs inference under the hood with no grad, so memory use is minimized.

Use smaller batch and sequence lengths: Ensure you are generating with batch size 1 (one prompt at a time) and not feeding an extremely long prompt that eats up memory. The context length of GPT-OSS is very large (up to 128k tokens), but you don’t want to max that out on a small GPU. For testing, keep prompts reasonably short (a few hundred tokens). The memory usage scales with sequence length, so a shorter prompt + generated output will use less memory.

Optional performance tweaks: If you installed xformers, you can activate its memory-efficient attention: model.enable_memory_efficient_attention() or use torch.set_float32_matmul_precision('high') for slightly more accurate matmuls in fp16. OpenAI’s GPT-OSS models also support FlashAttention 3 for faster inference on Hopper GPUs – though with an 11GB GPU (likely not an H100), this may not be applicable. In general, these optimizations affect speed more than memory, but faster kernels can indirectly reduce chance of memory fragmentation or spikes.


By following the above steps, you should be able to run inference on GPT-OSS (or similar 7B models) within the 11 GB VRAM limit. In summary, 4-bit quantization and Accelerate’s offloading are the key to low-memory inference.

6. LoRA/QLoRA Fine-Tuning Readiness

The setup we’ve prepared is LoRA-friendly. In fact, loading a model in 4-bit with bitsandbytes is exactly what you need for QLoRA (Quantized LoRA) training. A few notes to ensure smooth fine-tuning later:

PEFT integration: The PEFT library supports attaching LoRA adapters to Hugging Face models. You can take the model we loaded and prepare it for training with LoRA. For example, using PEFT’s get_peft_model and LoraConfig to wrap the model. Since we called model = AutoModelForCausalLM.from_pretrained(..., load_in_4bit=True), the base model’s weights are already frozen in 4-bit and cannot be updated (which is fine – QLoRA will not update base weights, only the LoRA layers). The PEFT library will handle keeping the gradients only for the small LoRA layers (which are in float32/16 and well within 11GB limits).

Gradient Checkpointing: For larger models or longer sequences, enabling gradient checkpointing during fine-tuning can reduce memory usage by trading compute for memory. Transformers supports model.gradient_checkpointing_enable(). This is something to keep in mind if you fine-tune the 20B model on 11GB GPU – you will likely need gradient checkpointing and a low batch size (as shown by HF experiments where Llama-7B and 13B could be fine-tuned on a 16GB T4 with gradient checkpointing + 4-bit). For a 7B-scale model, you might get away without checkpointing if batch size is 1.

Optimizer: Use an 8-bit optimizer or CPU offloaded optimizer to save memory. The QLoRA paper introduced the 4-bit Adam optimizer, but an easier route is to use Hugging Face’s bitsandbytes.optim.AdamW8bit (8-bit Adam) for training the LoRA parameters, since the LoRA layers are small.

Example fine-tune procedure: When ready to fine-tune, you’d do something like:

from peft import LoraConfig, get_peft_model
lora_config = LoraConfig( 
    r=16, lora_alpha=32, target_modules=["q_proj","v_proj"], 
    lora_dropout=0.05, bias="none", task_type="CAUSAL_LM" 
)
model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# -> This will show only a few trainable params (LoRA layers), others frozen.

Then proceed with your training loop or Trainer API. The base model remains in 4-bit mode (frozen) and the LoRA layers are float32. This setup is exactly what QLoRA paper uses – backprop through a 4-bit quantized model into LoRA adapters.

Memory during training: Monitor GPU memory during fine-tuning. 11 GB is limited, so you may need to reduce batch size, use gradient accumulation, and so on. The 7B with LoRA on 11GB is feasible (community reports show LLaMA-7B QLoRA training on a 12GB card works). Our use of FP16 compute and device_map offloading will continue to apply during training as well – Accelerate will keep some layers on CPU if they were offloaded. This can slow down training but is necessary to fit. If you find training too slow due to offloading, consider using only a 7B dense model (which should fully fit in 11GB in 4-bit without offload).


In short, this environment is prepared for QLoRA – the model is loaded in 4-bit, and PEFT can be used to fine-tune a LoRA adapter on top. OpenAI’s model card explicitly states GPT-OSS is fine-tunable, and the recommended approach for 4-bit models is using adapters rather than full-weight training.

7. Compatibility Caveats and Known Issues

Finally, let’s highlight a few caveats and fixes for potential issues given your specific versions (Python 3.12, CUDA 11.8, cuDNN 9.0):


作成日時: 2025-08-06 08:21:10