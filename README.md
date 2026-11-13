# invoice-email-resume-json-v1

Fine-tuning a small open model (Qwen3-1.7B) to extract structured JSON from plain-text **invoices, emails and resumes**. This repo holds the evaluation logs and results for the project.

## Results

Evaluated on `test_heldout` (1,500 documents), same scoring for both models.

| Metric | Base Qwen3-1.7B | Fine-tuned (LoRA v1) |
|---|---|---|
| Lenient field F1 | 0.6496 (CI 0.638-0.661) | **0.9800** (CI 0.9765-0.9832) |
| Document exact match | 1.3% | **82.1%** |
| Valid JSON | 99.9% | **100%** |
| Null hallucination | 78% | **1.4%** |

Per-domain field F1:

| Domain | Base | Fine-tuned |
|---|---|---|
| Invoice | 0.81 | 0.9953 |
| Email | 0.57 | 0.9872 |
| Resume | 0.51 | 0.9599 |

The 95% accuracy target was met.

**Known weak spots:** `email.meeting.start` (F1 0.695), `resume.location` (F1 0.849), and the two-column resume layout (F1 0.9266, document exact match 37.2%). The base model scored 0.0 on `email.date` and `email.meeting.start`.

## Dataset

Code-generated (no scraped data), with a verbatim filter that drops examples whose extracted values do not appear in the source text (405 examples dropped).

| Split | Size |
|---|---|
| train | 30,000 |
| validation | 1,500 |
| test_heldout | 1,500 |

- Mix: 40% invoice, 30% email, 30% resume
- Max token length (Qwen3-1.7B tokenizer): 1024 invoice, 1024 email, 2048 resume
- Schemas and a normalization rulebook are defined per domain (invoice, email, resume)
- Hugging Face dataset: `pmrccs/invoice-email-resume-json-v1` (private, not publicly available)

## Model

| Item | Value |
|---|---|
| Base model | Qwen3-1.7B (`unsloth/Qwen3-1.7B`) |
| Method | LoRA SFT with Unsloth |
| Hardware | Kaggle 2x T4, DDP |
| Training | 1,876 steps, about 3 hours |
| Best checkpoint | step 1500 (lowest eval_loss, 0.00268) |
| Chat template | Plain ChatML, no think tags (works with llama.cpp, Ollama and GGUF) |

- LoRA adapter (safetensors): [`pmrccs/qwen3-1.7b-invoice-email-resume-json-lora-v1`](https://huggingface.co/pmrccs/qwen3-1.7b-invoice-email-resume-json-lora-v1)
- GGUF: [`pmrccs/qwen3-1.7b-invoice-email-resume-json-lora-v1-gguf`](https://huggingface.co/pmrccs/qwen3-1.7b-invoice-email-resume-json-lora-v1-gguf)
  - Quants: Q8_0, Q4_K_M, IQ4_XS, plus the BF16 master and imatrix
  - The adapter was trained on `unsloth/Qwen3-1.7B` and merged onto `Qwen/Qwen3-1.7B` for GGUF. Both repos have the same main weights, so the result is equivalent.

## Repo layout

<!-- Adjust to match the actual folders in this repo -->
```
.
├── README.md
├── base/          # base model generations and scores
└── finetuned/     # fine-tuned model generations and scores
```

## Method

1. Generate the dataset with code and push it to a private HF dataset repo (not released).
2. Evaluate the base model on `test_heldout` and log generations and scores here.
3. Fine-tune with LoRA SFT, keeping the best checkpoint by eval_loss.
4. Evaluate the fine-tuned model the same way and log results here.
5. Merge the adapter and export GGUF quants.

The training and evaluation notebooks are not published. Compute was Kaggle's free tier.

## Notes

- The model repos (LoRA safetensors and GGUF) and this GitHub repo are public. The dataset is private.
- Scoring uses lenient field-level F1 (with bootstrap confidence intervals), document exact match, JSON validity and null hallucination rate.

## Author

[pmrccs](https://linktr.ee/pmrccs)
