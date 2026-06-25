# TakeMeter

A fine-tuned text classifier that sorts r/worldcup posts by **what kind of take
they are**

## Planned stack

| Component          | Tool                                       |
| ------------------ | ------------------------------------------ |
| Base model         | `distilbert-base-uncased` (HuggingFace)    |
| Fine-tuning        | Google Colab (free T4 GPU)                 |
| Libraries          | `transformers`, `datasets`, `scikit-learn` |
| Zero-shot baseline | Groq `llama-3.3-70b-versatile`             |
