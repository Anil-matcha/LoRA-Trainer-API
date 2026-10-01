# LoRA Trainer API: comparisons and examples

Train image-generation LoRA adapters through Muapi’s unified API. This repository is a practical reference for choosing a trainer, preparing a dataset, submitting a job, and polling the result. It contains documentation and copy-paste API examples rather than a separate client library.

[Open the LoRA Trainer API landing page](https://muapi.ai/lora-trainer) · [Get an API key](https://muapi.ai/access-keys) · [API reference](https://muapi.ai/docs/api-reference)

## Related Projects

- [Open Generative AI](https://github.com/Anil-matcha/Open-Generative-AI) — curated open generative-AI tools and resources.
- [FLUX 3 Dev API](https://github.com/Anil-matcha/Flux-3-Dev-API) — related FLUX generation examples.
- [Awesome AI Image Models](https://github.com/Anil-matcha/awesome-ai-image-models) — image-model discovery and comparison.
- [Generative Media Skills](https://github.com/SamurAIGPT/muapi-skills) — reusable generative-media workflows.

## What this repository covers

- A comparison of the live Flux, FLUX.2 Klein, Qwen-Image, and Z-Image trainer endpoints.
- Dataset preparation, trigger words, training-style selection, and common validation mistakes.
- `curl` and Python examples for submitting a training job and polling `/predictions/{id}/result`.
- Payload notes for dataset data URLs, training steps, learning rate, LoRA rank, and trigger words.

## Trainer comparison

| Endpoint | Base family | Best starting point | Main inputs |
|---|---|---|---|
| `flux-lora-trainer` | FLUX.1 Dev | General-purpose character or style adapters | `images_data_url`, `training_style`, `trigger_phrase` |
| `flux-1-dev-style-lora-trainer` | FLUX.1 Dev | Style-focused training | `dataset`, `learning_rate`, `training_steps`, `trigger_word` |
| `flux-2-klein-4b-style-lora-trainer` | FLUX.2 Klein 4B | A lighter style-training workflow | `dataset`, `learning_rate`, `training_steps`, `trigger_word` |
| `flux-2-klein-9b-style-lora-trainer` | FLUX.2 Klein 9B | Higher-capacity FLUX.2 style training | `dataset`, `learning_rate`, `training_steps`, `trigger_word` |
| `qwen-image-lora-trainer` | Qwen-Image | Qwen image adapters | `data`, `learning_rate`, `lora_rank`, `steps`, `trigger_word` |
| `qwen-image-2512-lora-trainer` | Qwen-Image 2512 | Qwen 2512-specific adapters | `data`, `learning_rate`, `lora_rank`, `steps`, `trigger_word` |
| `z-image-lora-trainer` | Z-Image | Z-Image adapter training | `data`, `learning_rate`, `lora_rank`, `steps`, `trigger_word` |
| `z-image-base-lora-trainer` | Z-Image Base | Base-checkpoint Z-Image training | `data`, `learning_rate`, `lora_rank`, `steps`, `trigger_word` |

Pricing and availability can change. Use the [live Muapi page](https://muapi.ai/lora-trainer) and the model response as the source of truth before launching a batch.

## Quick start with curl

Create an API key at [muapi.ai/access-keys](https://muapi.ai/access-keys), then prepare a dataset zip. The trainer accepts the dataset in the field documented for the selected endpoint; this example encodes the zip as a data URL for `flux-lora-trainer`.

```bash
export MUAPI_API_KEY="your_key"
export DATASET_DATA_URL="data:application/zip;base64,$(base64 < dataset.zip | tr -d '\n')"

curl -X POST "https://api.muapi.ai/api/v1/flux-lora-trainer" \
  -H "x-api-key: $MUAPI_API_KEY" \
  -H "Content-Type: application/json" \
  -d "$(python -c 'import json,os; print(json.dumps({"images_data_url":os.environ["DATASET_DATA_URL"],"training_style":"character","trigger_phrase":"TOK_CHARACTER"}))')"
```

The submission response contains a request identifier. Poll the result endpoint until its status is `succeeded` or `completed`:

```bash
export REQUEST_ID="id_from_the_submission_response"
curl "https://api.muapi.ai/api/v1/predictions/$REQUEST_ID/result" \
  -H "x-api-key: $MUAPI_API_KEY"
```

See [`examples/quickstart.py`](examples/quickstart.py) for a complete polling loop.

## Dataset and payload guidance

- Use a consistent subject or visual style and remove unrelated images before training.
- Keep the trigger phrase unique; use the same phrase in later generation prompts.
- Start with the trainer’s documented defaults, then change one training parameter at a time.
- Store the returned adapter URL with the model, trigger phrase, dataset version, and training settings.
- Treat uploaded images and generated adapters as sensitive creative assets; do not commit them or API keys.

## Common failure modes

| Symptom | Check |
|---|---|
| Request rejected before training | Dataset field, data-URL prefix, zip validity, and JSON escaping. |
| Adapter produces weak identity/style | Caption consistency, image variety, and trigger phrase usage. |
| Job appears stuck | Keep polling the result endpoint; record the request ID and retry only after a terminal failure. |
| Output cannot be used later | Persist the result URL before temporary output storage expires. |

## License

The examples and documentation in this repository are released under the MIT License. The underlying models and generated assets remain subject to their own terms.
