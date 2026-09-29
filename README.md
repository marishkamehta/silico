# Silico

Silico is a small Python client that lets a language model sit in the agent
seat of a behavioral experiment. Your experiment loop presents a turn, Silico
returns a reply, and the same code can run against a hosted model, a local
OpenAI-compatible server, or an offline mock.

## Install

Use Python 3.10 or later.

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

On Windows PowerShell, create the environment with `py -m venv .venv` and
activate it with `.venv\Scripts\Activate.ps1`; then use the same `python`
commands.

## Quick Start

Run entirely offline with the built-in mock:

```python
from silico import make_llm

llm = make_llm("mock")
print(llm.respond("Choose A or B. Reply with only the letter."))
```

Run the included smoke check:

```bash
python smokes/smoke_chat.py --model mock
python examples/growing_trajectory.py
python -m pytest tests
```

## Model Registry

The model registry is a YAML file that maps a short name, such as
`vllm-local`, to the details needed to connect to a model. Copy the example
and edit the entries you use. The working `model-registry.yaml` file is
ignored by Git.

```bash
cp model-registry.yaml.example model-registry.yaml
```

Silico looks for the registry in this order:

1. The path in the `SILICO_MODELS` environment variable, if set.
2. `model-registry.yaml` in the current working directory.

To use a file elsewhere, set its path before starting Python:

```bash
export SILICO_MODELS="/path/to/model-registry.yaml"
```

On Windows PowerShell:

```powershell
Copy-Item model-registry.yaml.example model-registry.yaml
$env:SILICO_MODELS = "C:\path\to\model-registry.yaml"
```

Each entry supports these fields:

| Field | Meaning |
| --- | --- |
| `backend` | Serving backend label, such as `vllm`, `ollama`, or `azure`. |
| `base_url` | The OpenAI-compatible API root. |
| `served_name` | The model or deployment name expected by the endpoint. Defaults to the entry's short name if omitted. |
| `key_env` | Optional name of the environment variable containing the API key. Store the key in that variable, not in the registry. |
| `config` | Optional configuration kind. Omit it to use the default `chat` configuration. |

For example:

```yaml
vllm-local:
  backend: vllm
  base_url: http://localhost:8000/v1
  served_name: Qwen/Qwen2.5-3B-Instruct
```

Use the short name to create the client:

```python
llm = make_llm("vllm-local")
```

The built-in `mock` model is always available, needs no registry file, and
cannot be overridden. If no registry is configured or found at the default
location, only `mock` is available. A path explicitly set through
`SILICO_MODELS` must point to an existing file.

## License and Citation

Silico is released under the Creative Commons Attribution-NonCommercial 4.0
International license. Citation metadata is in [`CITATION.cff`](CITATION.cff).
