# Yibu API Examples and Usage Reporting

Use the [Python examples package](../examples/yibuapi_examples_20260918_v01.tar.gz?raw=true) in this repository to get started with Yibu API calls and record the token usage returned by the API. Your API key is delivered privately by email with a link to this guide. To return your usage summaries, reply to the email that delivered your API key and attach the reports by **September 20, 2026, 11:59 PM EDT (America/Toronto)**, the end of the event day in Waterloo.

The instructions below apply to `yibuapi_examples_20260918_v01.tar.gz`. Download it using the link above; if GitHub opens the archive's file page, choose **Download raw file** to save the archive.

Refer to your key-delivery email for your allocation details and to the organizers' subsequent notice for the planned top-up timing.

Submit only one application per team. Enter the applicant's name in the Name field and list all other members in Team Members. The allocation covers all of those people; a person already covered by an allocation must not request another one as an applicant or as a member of another application.

## 1. Install and configure the examples

In a terminal, change to the directory containing the downloaded archive, then run these macOS/Linux commands:

```bash
tar -xzf yibuapi_examples_20260918_v01.tar.gz
cd yibuapi_examples_20260918_v01
python3 -m venv .venv
. .venv/bin/activate
python -m pip install -r requirements.txt
export YIBU_API_KEY='PASTE_THE_KEY_FROM_YOUR_APPROVAL_EMAIL'
```

Replace the placeholder with your assigned key in your own terminal. Keep the key private. The scripts read `YIBU_API_KEY` from the process environment; they **do not automatically load `.env` or `.env.example`**. Creating an `.env` file alone will not configure them. The dependencies are `httpx` and `websockets`, with version bounds in `requirements.txt`.

## 2. Make a call and label its purpose

Run commands from inside the extracted package directory with the virtual environment activated:

```bash
python qwen35_omni_flash.py \
  --prompt 'Introduce yourself in one sentence.' \
  --purpose 'prototype_setup'

python qwen35_omni_plus.py \
  --prompt 'Describe this image.' \
  --image /path/to/example.jpg \
  --purpose 'image_understanding'
```

Use a short, consistent `--purpose` label, such as `prototype_setup`, `audio_understanding`, or `demo`, so summaries can group calls by use. Do not place prompts, personal information, or credentials in the label.

The package also includes these entry points:

| Script | Example route |
| --- | --- |
| `qwen35_omni_flash.py` | Qwen3.5 Omni Flash, HTTP Chat Completions |
| `qwen35_omni_plus.py` | Qwen3.5 Omni Plus, HTTP Chat Completions |
| `qwen35_omni_plus_realtime.py` | Qwen3.5 Omni Plus Realtime, WebSocket |
| `gemini31_flash_live.py` | Gemini 3.1 Flash Live, WebSocket |
| `chat_completions_generic.py` | Other compatible Chat Completions models; specify `--model` with an exact supported model ID |

Each entry point accepts `--purpose` and `--audit-log`. See the package README and each script's `--help` for options. These examples make real API calls and use the assigned credit; model access depends on your key and the provider.

## 3. Keep the local call ledger

Calls handled by the examples' API wrappers append success or failure records to:

```text
artifacts/yibu_api_calls.jsonl
```

This default path is relative to the package directory, even if you launch a script from another directory. You do not run `yibu_audit.py` as a separate monitoring process.

Records include a call ID, timestamps, model, key suffix, purpose, endpoint, transport, success/failure status, latency, and the returned usage information. The `key_suffix` field stores only the final four key characters, prefixed with `...`; it is not a complete key or a globally unique identifier.

The logger records normalized `input_tokens`, `output_tokens`, and `total_tokens` when the API provides them. It retains the original usage object as `usage_raw`. If the API omits a total but provides both input and output counts, the logger derives their sum and marks `total_tokens_derived`.

**Missing token values are unknown, not zero.** The ledger leaves missing values as `null`. Summaries add only reported values and include missing-value counters; a total of zero with missing calls does not establish zero consumption. The tools do not calculate monetary cost or remaining credit.

To use a persistent ledger at a different location, choose either the environment variable or the per-call option:

```bash
export YIBU_AUDIT_LOG='/path/to/private/yibu_api_calls.jsonl'

# Alternatively, override the location for a particular call:
python qwen35_omni_flash.py \
  --prompt 'Hello.' \
  --purpose 'prototype_setup' \
  --audit-log '/path/to/private/yibu_api_calls.jsonl'
```

The explicit `--audit-log` option takes precedence over `YIBU_AUDIT_LOG`. Keep track of every ledger your team uses so your report covers the relevant calls.

## 4. Generate usage summaries

For the default ledger, run:

```bash
python summarize_usage.py
```

The tool writes:

```text
artifacts/summary/usage_summary.json
artifacts/summary/usage_by_model_key_purpose.csv
```

For a custom ledger or output directory, provide both paths explicitly:

```bash
python summarize_usage.py \
  --log '/path/to/private/yibu_api_calls.jsonl' \
  --out-dir '/path/to/private/summary'
```

**The summary command does not use `YIBU_AUDIT_LOG` to select its input.** Pass `--log` whenever your calls wrote to a non-default ledger. The ledger must already exist.

The JSON file includes overall totals, grouped totals, missing-value counts, and source information. The CSV groups calls by model, key suffix, and purpose, with success/failure counts and token totals. The summary tool ignores identical duplicate call IDs and rejects conflicting records with the same ID.

## 5. Report to the organizers

Reply to the email that delivered your API key and attach these two summary files by **September 20, 2026, 11:59 PM EDT (America/Toronto)**, the end of the event day in Waterloo:

- `usage_summary.json`
- `usage_by_model_key_purpose.csv`

Include your team name, project/repository link, the email address used for the API application, the reporting period, and the key suffix or suffixes in the report message. The organizers can use the team name and application email to associate the report with your allocation; a four-character suffix alone may collide with another key.

Describe any missing usage, excluded calls, additional ledgers, or custom logging so the coverage is clear. If your team made no API calls, state that in your reply instead of fabricating a ledger. Preserve the JSONL ledger locally for follow-up reconciliation; send it privately only if the organizers request it.

Before sharing, inspect the files you will send. Although the logger does not intentionally store prompts, response bodies, or the complete API key, `usage_raw` contains provider-supplied data and `error` can contain sensitive details from failures. The JSON summary's `source.path` is an absolute local path and may reveal a username or internal directory. Inspect custom purpose labels and endpoints as well. Remove sensitive details from a separate sharing copy and note any redactions; keep the original ledger private for reconciliation.

Do not include a complete API key, `.env` file, raw prompts, or input/output media in a usage report or a public repository, issue, or pull request. Remove any quoted API key from your reply before sending. Do not post usage reports publicly.

## 6. Integrating the logger into your own application

Running these examples does not monitor calls made elsewhere. If your application uses its own SDK or client, explicitly integrate `append_audit_record` from `yibu_audit.py`, or produce equivalent records with the required fields. Record each relevant call and its returned usage, including failures, with a stable purpose label. Do not copy the complete API key into your own logging fields.

The packaged HTTP examples use non-streaming Chat Completions. The two WebSocket examples each run a single turn and record usage observed during that turn: the Qwen example uses the final `response.done` event, and the Gemini example retains the most recently received `usageMetadata` before the turn completes. They do not automatically provide complete accounting for arbitrary streaming, interrupted, retried, or multi-turn applications. When adapting them, verify whether the provider reports per-response or cumulative usage, retain missing-usage indicators, and avoid counting the same usage twice.
