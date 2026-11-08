# INFERON AI

An incident response agent that remembers past production incidents and writes response plans grounded in them.

Describe a live incident in a web form. INFERON AI recalls similar past incidents from memory, passes them to a language model, and returns a five-section plan. The plan quotes the memory it relied on, so you can check it against the source.

## What it does

- Stores past incidents, root causes, resolution steps, runbooks, and postmortem lessons in a Hindsight memory bank
- Recalls the most relevant memories for a new incident using Hindsight's search
- Writes a response plan with a fixed structure: likely cause, immediate steps, supporting memories, verification steps, and escalation criteria
- Saves new postmortems and resolved incidents back to memory, so later plans can use them
- Provides a Gradio web form for use during an incident

## Architecture

- **Memory:** [Hindsight](https://github.com/vectorize-io/hindsight) cloud API, using one memory bank named `incident-memory`
- **Model:** `openai/gpt-oss-120b` on Groq, used to write plans and extract postmortem lessons
- **Interface:** Gradio web form, shared through a temporary public link

## Setup

1. Open the notebook `inferon_ai.ipynb` in Google Colab.
2. Add two secrets in Colab's Secrets panel, with **Notebook access** turned on:
   - `HINDSIGHT_API_KEY`: your Hindsight Cloud API key
   - `GROQ_API_KEY`: your Groq API key
3. Run the cells in order:
   1. Setup (installs the libraries and connects to Hindsight and Groq)
   2. Seed data (defines the sample incidents; the storage loop is commented out)
   3. Optional: postmortem and resolution cells, only when you have new data to save
   4. Gradio cell (starts the web form and prints a public link)

## Usage

Enter an incident title, service, symptoms, and severity, then click **Get response plan**. The example below uses data from the seeded memory:

- **Title:** Checkout 503s after release v2.7
- **Service:** checkout-api
- **Symptoms:** 503 errors rising, pods restarting, memory growing steadily
- **Severity:** SEV1

## Known limits

- **Memory persists in the cloud.** Hindsight stores memories between Colab sessions. The seed data is stored once and should not be loaded again, or it will be duplicated.
- **Ranking favors the closest match.** When a memory outcome comes from a different service (for example, a payments rollback), the plan may not surface it for a checkout query.
- **Quotes are not always exact.** The model is asked to quote memory word for word. In testing, one plan paraphrased a memory while presenting it as a quote, so check citations against the source.
- **Severity has a small effect.** The severity field changes the escalation wording, but the steps are similar across SEV1, SEV2, and SEV3.
- **Plans are suggestions.** Check every step against your real runbooks before acting on it.

## Status

Working prototype. The seed incidents are examples, not production data. The public link works only while the Colab session is running.

## Tech stack

- Python
- Hindsight (`hindsight-client`)
- Groq API (`openai/gpt-oss-120b`)
- Gradio
