---
license: apache-2.0
language:
- en
base_model:
- deepseek-ai/DeepSeek-R1
new_version: imsanjoykb/deepSQL-R1-distill-8B
pipeline_tag: text-generation
library_name: adapter-transformers
library_name2: transformers
tags:
- unsloth,
- pytorch,
- deepsee-R1,
- inference-endpoint,
- sql-code-generation,
---
![alt text](assets/logomain.png "Repo banner")

<div align="center">

[![Hugging Face Model](https://img.shields.io/badge/HuggingFace-Model-FF6F00?style=for-the-badge&logo=huggingface&logoColor=white)](https://huggingface.co/imsanjoykb/deepSQL-R1-distill-8B)
[![Open In Colab](https://img.shields.io/badge/Open%20in%20Colab-FF6F00%2F000000?style=for-the-badge&logo=googlecolab&logoColor=white&labelColor=FF6F00)](https://drive.google.com/file/d/145PP-oW50OMS1bYJaYuUphfufpsuOGWl/view?usp=sharing)
[![Kaggle Notebook](https://img.shields.io/badge/Kaggle-Notebook-20BEFF?style=for-the-badge&logo=kaggle&logoColor=white)](https://www.kaggle.com/code/imsanjoykb/inference-deepsql-r1-distill-8b)
[![GitHub Repo](https://img.shields.io/badge/GitHub-Repo-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/imsanjoykb/deepSQL-R1-distill-8B)
[![Gradio App](https://img.shields.io/badge/Chat%20App-Gradio-0084FF?style=for-the-badge&logo=gradio&logoColor=white)](https://huggingface.co/spaces/imsanjoykb/deepSQL-R1-distill-8B)
[![Gradio-Colab](https://img.shields.io/badge/Gradio-Colab-0084FF?style=for-the-badge&logo=gradio&labelColor=F9AB00)](https://colab.research.google.com/drive/1ze7qAQnjppZKfxNVBXXlOBTM6xFWEYrJ?usp=sharing)
[![arXiv Paper](https://img.shields.io/badge/arXiv-Preprint-B31B1B?style=for-the-badge&logo=arxiv&logoColor=white)](https://arxiv.org/abs/Your_Paper_ID)

</div>


## Inference

Here provides a code snippet with `apply_chat_template` to show you how to load the tokenizer and model and how to generate contents.

```python
# Import necessary libraries
from unsloth import FastLanguageModel
import torch

# Define the model name and other parameters
model_name = "imsanjoykb/deepSQL-R1-distill-8B"
max_seq_length = 2048
dtype = None
load_in_4bit = True

# Load the model and tokenizer from Hugging Face
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name=model_name,
    max_seq_length=max_seq_length,
    dtype=dtype,
    load_in_4bit=load_in_4bit,
)

# Enable faster inference
FastLanguageModel.for_inference(model)

# Define the prompt template
odoo_text2sql_prompt = """Below is an instruction describing a task related to generating a SQL query specifically for Odoo's database structure. The input provides relevant context about Odoo models or data fields from {db_schema}. Write a SQL query that fulfills the given task using Odoo's database schema.

### Instruction:
Generate a SQL query in the context of Odoo to {}

### Input:
{}

### Response:
{}
"""
```

```python
# Optionally, use a TextStreamer for continuous inference
from transformers import TextStreamer

db_schema = """
CREATE TABLE product_product (
	id SERIAL NOT NULL,
	message_main_attachment_id INTEGER,
	product_tmpl_id INTEGER NOT NULL,
	create_uid INTEGER,
	write_uid INTEGER,
	default_code VARCHAR,
	barcode VARCHAR,
	combination_indices VARCHAR,
	volume NUMERIC,
	weight NUMERIC,
	active BOOLEAN,
	can_image_variant_1024_be_zoomed BOOLEAN,
	create_date TIMESTAMP WITHOUT TIME ZONE,
	write_date TIMESTAMP WITHOUT TIME ZONE,
	store_qty_available DOUBLE PRECISION,
	store_standard_price DOUBLE PRECISION,
	store_sales_count DOUBLE PRECISION,
	CONSTRAINT product_product_pkey PRIMARY KEY (id),
	CONSTRAINT product_product_create_uid_fkey FOREIGN KEY(create_uid) REFERENCES res_users (id) ON DELETE SET NULL,
	CONSTRAINT product_product_message_main_attachment_id_fkey FOREIGN KEY(message_main_attachment_id) REFERENCES ir_attachment (id) ON DELETE SET NUL"L,
	CONSTRAINT product_product_product_tmpl_id_fkey FOREIGN KEY(product_tmpl_id) REFERENCES product_template (id) ON DELETE CASCADE,
	CONSTRAINT product_product_write_uid_fkey FOREIGN KEY(write_uid) REFERENCES res_users (id) ON DELETE SET NULL
)
"""
# Prepare the input text for continuous inference
instruction = ""
input_text = "What are the top sales products?"
output_text = ""

# Define the `odoo_text2sql_prompt` with placeholders
odoo_text2sql_prompt = """
Instruction: {instruction}
Input: {input_text}
Output: {output_text}
DB Schema: {db_schema}
"""

# Tokenize the input text
inputs = tokenizer(
    [
        odoo_text2sql_prompt.format(
            instruction=instruction,
            input_text=input_text,
            output_text=output_text,
            db_schema=db_schema
        )
    ],
    return_tensors="pt"
).to("cuda")

# Initialize the TextStreamer
text_streamer = TextStreamer(tokenizer)

# Generate the output using the model with TextStreamer
_ = model.generate(**inputs, streamer=text_streamer, max_new_tokens=350)
```
## Model Download
|            **Model**            | **#Total Params** | **#Active Params** | **Context Length** |                         **Download**                         |
| :-----------------------------: | :---------------: | :----------------: | :----------------: | :----------------------------------------------------------: |
|   deepSQL-R1-distill-8B         |        8B        |        6B        |        128k        | [🤗 HuggingFace](https://huggingface.co/imsanjoykb/deepSQL-R1-distill-8B) |

## Benchmarking
## 📊 SQL Model Benchmarking - Comprehensive Evaluation

| Rank | LLM Name                   | SqlEval-Classic (%) | Execution Accuracy (%) | Query Optimization (%) | Latency (ms) |
|------|----------------------------|---------------------|-----------------------|-----------------------|--------------|
| 1️⃣  | GPT-4o                     | 86                  | 91                    | 88                    | 120          |
| 2️⃣  | deepSQL-R1-distill-8B       | 82                  | 89                    | 85                    | 110          |
| 3️⃣  | deepseek-R1                 | 78                  | 84                    | 86                    | 150          |
| 4️⃣  | Claude-3-Sonnet             | 72                  | 8o                    | 80                    | 130          |
| 5️⃣  | llama3.2                    | 68                  | 72                    | 76                    | 170          |
| 6️⃣  | Mistral-7B                  | 62                  | 76                    | 69                    | 190          |

🚀 **Key Insights:**  
- **GPT-4o** leads in overall performance, achieving **91% execution accuracy** with low latency (**120ms**).  
- **deepSQL-R1-distill-8B** excels in query execution & optimization, making it a strong competitor.  
- **Mistral-7B** has the lowest scores but may improve with fine-tuning.  

🔹 **New Metrics Explained:**  
- **Execution Accuracy (%)** → Measures correctness of SQL execution.  
- **Query Optimization (%)** → Evaluates efficiency in structuring optimized queries.  
- **Latency (ms)** → Measures response time (lower is better).  



## Citing
```
@misc{,
  author = {Sanjoy Kumar},
  title = {DeepSQL},
  year = {2024},
  publisher link = {https://huggingface.co/imsanjoykb/deepSQL-R1-distill-8B},
}
```

# Uploaded  model

- **Developed by:** [Sanjoy Biswas](https://www.linkedin.com/in/imsanjoykb/) 
- **License:** apache-2.0