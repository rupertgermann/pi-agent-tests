# Qwen3.8-27B-GGUF llama.cpp commands

## Georgi Gerganov @ggerganov

### simple:
```bash
llama serve -hf ggml-org/Qwen3.8-27B-GGUF --spec-type draft-mtp
```

### advanced:
```bash
llama serve \
  -hf ggml-org/Qwen3.8-27B-GGUF \
  --spec-default \
  --spec-type draft-mtp \
  --reasoning-preserve --agent
```

### 32GB VRAM (e.g. RTX 5090):
```bash

llama serve \
  -hf  ggml-org/Qwen3.8-27B-GGUF:Q4_K_M \
  -hfd ggml-org/Qwen3.8-27B-GGUF:Q4_0 \
  --spec-default \
  --spec-type draft-mtp \
  --ctx-size 196608 \
  --cache-type-k q8_0 \
  --cache-type-v q8_0 \
  --reasoning-preserve --fit off --agent
```

### on DGX Spark:
```bash
llama serve \
 -hf  ggml-org/Qwen3.8-27B-GGUF:Q4_K_M \
 -hfd ggml-org/Qwen3.8-27B-GGUF:Q4_0 \
 --spec-default \
 --spec-type draft-mtp \
 --reasoning-preserve --agent
```

### to limit the max reasoning length:
```bash
... \
--reasoning-budget 4096 \
--reasoning-budget-message "... I am thinking for too -- let me gather more info about the task."
```

adjust to your needs



## Sebastian @Danmoreng

### 16GB RTX 5080 Laptop:
```bash
llama serve \
  -hf unsloth/Qwen3.8-27B-GGUF \
  -hff Qwen3.8-27B-UD-IQ3_XXS.gguf \
  --no-mmproj -c 65536 \
  -ctk q8_0 -ctv q8_0 \
  -b 1024 -np 1 \
  --spec-default --spec-type draft-mtp \
  --reasoning-preserve --fit off --agent
```

~60 t/s.

---


## prompt

write a solar system simulation that i can run in my browser. single html file. no dependencies.


### current llama.cpp command

```bash
llama serve \
  --models-dir ~/llama-models \
  --no-models-autoload \
  --models-max 1 \
  --jinja \
  --host 127.0.0.1 \
  --port 8080 \
  -ngl 999 \
  --ctx-size 128000 \
  --parallel 1 \
  --cache-type-k q8_0 \
  --cache-type-v q8_0 \
  -hfd ggml-org/Qwen3.8-27B-GGUF:Q4_0 \
  --spec-default \
  --spec-type draft-mtp \
  --reasoning-preserve \
  --fit off
```


--> 11 tokens/s


---

### next llama.cpp command

```bash
llama serve \
  --models-dir ~/llama-models \
  --no-models-autoload \
  --models-max 1 \
  --jinja \
  --host 127.0.0.1 \
  --port 8080 \
  -ngl 999 \
  --flash-attn on \
  --ctx-size 131072 \
  --parallel 1 \
  --no-mmproj \
  --cache-type-k f16 \
  --cache-type-v f16 \
  -hfd ggml-org/Qwen3.8-27B-GGUF:Q4_0 \
  --spec-default \
  --spec-type draft-mtp \
  --spec-draft-n-max 2 \
  --reasoning-budget 2048 \
  --fit off

  ````
--> 16 tokens/s


### next next llama.cpp command

```bash
llama serve \
  --models-dir "$HOME/llama-models" \
  --no-models-autoload \
  --models-max 1 \
  --jinja \
  --host 127.0.0.1 \
  --port 8080 \
  -ngl all \
  --spec-draft-ngl all \
  --flash-attn on \
  --ctx-size 131072 \
  --parallel 1 \
  --no-mmproj \
  --cache-type-k f16 \
  --cache-type-v f16 \
  -hfd ggml-org/Qwen3.8-27B-GGUF:Q4_0 \
  --spec-default \
  --spec-type draft-mtp \
  --spec-draft-n-max 3 \
  --reasoning on \
  --reasoning-budget 1024 \
  --no-reasoning-preserve \
  --fit off

```
--> 13 tokens/s


