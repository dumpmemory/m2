# Long-Context Retrieval with M2-BERT

We're excited to release some new long-context M2-BERT models (2k, 8k, 32k) as well as embedding versions fine-tuned for long-context retrieval!
As part of this release, we're also releasing a preview of LoCo, a new benchmark for long-context retrieval.

* [Setup](#setup)
* [Obtaining Pretrained Checkpoints](#obtaining-pretrained-checkpoints)
* [Generating Embeddings](#generating-embeddings)
* [Running LoCo](#running-loco)

## Setup

Follow the same setup as in the general [README](README.md).
We recommend having `flash-fft-conv` installed:

```bash
pip install git+https://github.com/HazyResearch/flash-fft-conv.git#subdirectory=csrc/flashfftconv
pip install git+https://github.com/HazyResearch/flash-fft-conv.git
```

If you don't have it installed, you can still run the code, but you will need to set `use_flashfft` to `False`` in the `yamls/embeddings` files.
Model loading will print out that you are missing some parameters, but that is fine.

For inference, you additionally need to install the dependencies in [requirements-embeddings.txt](requirements-embeddings.txt).
This is mostly `beir` and a few extra libraries to run embedding models that are only available behind existing APIs.

## Obtaining Pretrained Checkpoints

You can download pretrained checkpoints from HuggingFace:
* [M2-BERT-base-80M-2K-retrieval](https://huggingface.co/togethercomputer/m2-bert-80M-2k-retrieval)
* [M2-BERT-base-80M-8K-retrieval](https://huggingface.co/togethercomputer/m2-bert-80M-8k-retrieval)
* [M2-BERT-base-80M-32K-retrieval](https://huggingface.co/togethercomputer/m2-bert-80M-32k-retrieval)

## Generating Embeddings

You can see [embed_text.py](embed_text.py) for a minimal example of generating embeddings using the M2 BERT models.
We do not recommend using this to process many documents, since it runs with batch size 1.

```bash
python embed_text.py --text "hello world" --model-name togethercomputer/m2-bert-80M-2k-retrieval --yaml-file yamls/embeddings/m2-bert-80M-2k-retrieval.yaml

python embed_text.py --text "hello world" --model-name togethercomputer/m2-bert-80M-8k-retrieval --yaml-file yamls/embeddings/m2-bert-80M-8k-retrieval.yaml

python embed_text.py --text "hello world" --model-name togethercomputer/m2-bert-80M-32k-retrieval --yaml-file yamls/embeddings/m2-bert-80M-32k-retrieval.yaml
```

Or using the Together API (you can find your API key [here](https://api.together.xyz/settings/api-keys)):

```bash
export TOGETHER_API_KEY={YOUR API KEY HERE}

python embed_text.py --text "hello world" --model-name togethercomputer/m2-bert-80M-2k-retrieval --together-api

python embed_text.py --text "hello world" --model-name togethercomputer/m2-bert-80M-8k-retrieval --together-api

python embed_text.py --text "hello world" --model-name togethercomputer/m2-bert-80M-32k-retrieval --together-api
```

You can use the Together API to generate embeddings from any of these models by querying directly:
```Python
import os
import requests

def generate_together_embeddings(text: str, model_api_string: str, api_key: str):
    url = "https://api.together.xyz/api/v1/embeddings"
    headers = {
        "accept": "application/json",
        "content-type": "application/json",
        "Authorization": f"Bearer {api_key}"
    }
    session = requests.Session()
    response = session.post(
        url,
        headers=headers,
        json={
            "input": text,
            "model": model_api_string
        }
    )
    if response.status_code != 200:
        raise ValueError(f"Request failed with status code {response.status_code}: {response.text}")
    return response.json()['data'][0]['embedding']

print(generate_together_embeddings('Hello world', 'togethercomputer/m2-bert-80M-32k-retrieval', os.environ['TOGETHER_API_KEY'])[:10])
```

## Running LoCo

You can use these commands to evaluate M2-BERT on LoCo locally:

```bash
python loco_eval.py --model-name togethercomputer/m2-bert-80M-2k-retrieval --yaml-file yamls/embeddings/m2-bert-80M-2k-retrieval.yaml

python loco_eval.py --model-name togethercomputer/m2-bert-80M-8k-retrieval --yaml-file yamls/embeddings/m2-bert-80M-8k-retrieval.yaml

python loco_eval.py --model-name togethercomputer/m2-bert-80M-32k-retrieval --yaml-file yamls/embeddings/m2-bert-80M-32k-retrieval.yaml
```

Or using the Together API (you can find your API key [here](https://api.together.xyz/settings/api-keys)):

```bash
export TOGETHER_API_KEY={YOUR API KEY HERE}

python loco_eval.py --model-name togethercomputer/m2-bert-80M-2k-retrieval --together-api

python loco_eval.py --model-name togethercomputer/m2-bert-80M-8k-retrieval --together-api

python loco_eval.py --model-name togethercomputer/m2-bert-80M-32k-retrieval --together-api
```