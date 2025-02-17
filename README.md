#  XiYanSQL-QwenCoder-32B

### Important Links


🤖[ModelScope](https://www.modelscope.cn/models/XGenerationLab/XiYanSQL-QwenCoder-32B-2412) |
📖[XiYan-SQL](https://github.com/XGenerationLab/XiYan-SQL) |
🌕[析言GBI](https://bailian.console.aliyun.com/xiyan) |
🤗[Modelscope Space](https://www.modelscope.cn/studios/XGenerationLab/XiYanSQL-QwenCoder-32B)

HuggingFace linking is coming...

## Introduction
We open-source the first XiYanSQL-QwenCoder-32B model on January 22, 2025, and we look forward to contributing to the text-to-SQL community.
**XiYanSQL-QwenCoder-32B**, a SQL model fine-tuned on the Qwen2.5Coder-32B model, achieves an EX score of **69.03%** on the BIRD test set, setting a new SOTA under only a single fine-tuned model.
In the future, we will release more SQL-related models.


## Requirements

transformers >= 4.37.0

## Quickstart

> NOTE: XiYanSQL-QwenCoder-32B can be used directly for text-to-SQL tasks or serve as a better starting point for fine-tuning SQL models.


Here is a simple code snippet for quickly using **XiYanSQL-QwenCoder-32B** model. We provide a Chinese version of the prompt, and you just need to replace the placeholders for "question," "db_schema," and "evidence" to get started. We recommend using our [M-Schema](https://github.com/XGenerationLab/M-Schema) format for the schema; other formats such as DDL are also acceptable, but they may affect performance.
Currently, we mainly support mainstream dialects like SQLite, PostgreSQL, and MySQL.

```

nl2sqlite_template_cn = """你是一名{dialect}专家，现在需要阅读并理解下面的【数据库schema】描述，以及可能用到的【参考信息】，并运用{dialect}知识生成sql语句回答【用户问题】。
【用户问题】
{question}

【数据库schema】
{db_schema}

【参考信息】
{evidence}

【用户问题】
{question}

```sql"""

import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = "XGenerationLab/XiYanSQL-QwenCoder-32B-2412"
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype=torch.bfloat16,
    device_map="auto"
)

tokenizer = AutoTokenizer.from_pretrained(model_name)

## dialects -> ['SQLite', 'PostgreSQL', 'MySQL']
prompt = nl2sqlite_template_cn.format(dialect="", db_schema="", question="", evidence="")
message = [{'role': 'user', 'content': prompt}]

text = tokenizer.apply_chat_template(
    message,
    tokenize=False,
    add_generation_prompt=True
)
model_inputs = tokenizer([text], return_tensors="pt").to(model.device)

generated_ids = model.generate(
    **model_inputs,
    pad_token_id=tokenizer.pad_token_id,
    eos_token_id=tokenizer.eos_token_id,
    max_new_tokens=1024,
    temperature=0.1,
    top_p=0.8,
    do_sample=True,
)
generated_ids = [
    output_ids[len(input_ids):] for input_ids, output_ids in zip(model_inputs.input_ids, generated_ids)
]
response = tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]

```

## Performance
XiYanSQL-QwenCoder-32B, as a multi-dialect SQL base model, demonstrating robust SQL generation capabilities. The following presents the evaluation results at the time of release. We conducted a comprehensive evaluation of the model's performance under two schema formats, M-Schema, and original DDL, using the BIRD and Spider benchmarks in the Text-to-SQL domain.

| Model name|BIRD Dev@M-Schema |BIRD Dev@DDL|Spider Test@M-Schema|Spider Test@DDL|
|-----------|------------------|---------------|-------------------|---------------|
|Codellama-34b              | 33.05%     | -          | 65.88%      | -           |
|Deepseek-coder-33b         | 47.52%     | 44.72%     | -           | 70.22%      |
|TableGPT2                  | 46.35%     | 47.07%     | 72.89%      | 74.41%      |
|Codestral 22b              | 50.52%     | 47.00%     | 76.12%      | 72.39%      |
|Claude35_sonnet-1022       | 53.32%     | -          | 73.65%      | -           |
|GLM-4-plus                 | 54.37%     | -          | 77.28%      | -           |
|Deepseek(v2.5-1210)        | 55.74%     | 55.61%     | 79.86%      | 77.54%      |
|Gemini-1.5-pro             | 61.34%     | -          | 82.84%      | -           |
|GPT-4o-0806                | 58.47%     | 54.82%     | 80.41%      | 75.77%      |
|**XiYanSQL-QwenCoder-32B** | **67.01%** | **63.04%** | **86.52%**  | **83.44%**  |


## Acknowledgments
If you find our work useful, please give us a citation or a star, so we can make a greater contribution to the open-source community!











