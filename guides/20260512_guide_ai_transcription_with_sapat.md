---
title: 'AI Transcription with Sapat in Daytona'
description:
  'Set up Sapat in a Daytona workspace and transcribe audio with OpenAI, Groq, or Azure OpenAI Whisper APIs.'
date: 2026-05-12
author: 'AI Agent'
tags: ['ai', 'transcription', 'daytona', 'sapat', 'openai', 'groq']
---

# AI Transcription with Sapat in Daytona

# Introduction

Audio and video content is everywhere in engineering teams: product demos,
customer calls, user interviews, internal training sessions, conference talks,
and debugging recordings. The hard part is not recording that material. The hard
part is turning it into text that can be searched, summarized, quoted, reviewed,
and reused by the rest of the team.

[Sapat](https://github.com/nkkko/sapat) is a small Python transcription tool that
wraps multiple Whisper-compatible providers behind one command-line interface.
It can send audio to OpenAI, Groq Cloud, or Azure OpenAI, and it can optionally
run a chat model over the transcript to correct punctuation and spelling. When
combined with a [Daytona workspace](/definitions/20240819_definition_daytona%20workspace.md),
Sapat becomes a reproducible transcription environment: dependencies, API
configuration, and output files all live in the same development workspace.

This guide walks through a practical setup. You will clone Sapat in Daytona,
install its dependencies, configure provider credentials, run transcriptions with
OpenAI, Groq, and Azure OpenAI, and verify the generated transcript files.

## TL;DR

- Clone Sapat into a Daytona workspace.
- Install Python dependencies and make sure `ffmpeg` is available.
- Add one or more provider credentials to `.env`.
- Run `python -m src.sapat.script <file> --api openai|groq|azure`.
- Use `--language`, `--prompt`, `--temperature`, `--quality`, and `--correct` to
  tune the transcript.

## Prerequisites

Before you begin, prepare the following:

- A running Daytona workspace with terminal access.
- Python 3 and `pip` installed in the workspace.
- `ffmpeg`, because Sapat converts input media to MP3 before transcription.
- An audio or video file to transcribe. Short files are best for the first test.
- At least one API credential:
  - OpenAI API key for `whisper-1`.
  - Groq Cloud API key for `whisper-large-v3-turbo` or another supported Whisper
    model.
  - Azure OpenAI key, endpoint, deployment name, and API version.

The commands below assume a Linux shell inside Daytona.

## Step 1: Create the Sapat Workspace Setup

Start by cloning the repository into your Daytona workspace:

```bash
cd ~
git clone https://github.com/nkkko/sapat.git
cd sapat
```

Install the Python packages listed by the project:

```bash
python -m pip install -r requirements.txt
```

Sapat relies on `ffmpeg` for media conversion. Check whether it is already
available:

```bash
ffmpeg -version
```

If the command is missing in your workspace image, install it with your image's
package manager. For Debian or Ubuntu based images, use:

```bash
sudo apt-get update
sudo apt-get install -y ffmpeg
```

Keep a small sample file in the repository while testing. For example:

```bash
mkdir -p samples transcripts
cp /path/to/demo-recording.mp4 samples/demo-recording.mp4
```

## Step 2: Configure Provider Credentials

Sapat loads environment variables from a `.env` file in the project directory.
Copy the example file first:

```bash
cp .env.example .env
```

Open `.env` and fill in the provider you want to use. You can configure only one
provider, or configure all three and choose between them at runtime.

### OpenAI Whisper

Use OpenAI when you want the standard hosted Whisper API and a simple setup:

```bash
OPENAI_API_KEY=your_openai_api_key
OPENAI_MODEL=whisper-1
OPENAI_API_ENDPOINT=https://api.openai.com/v1/audio/transcriptions
OPENAI_MODEL_NAME_CHAT=gpt-4o
```

`OPENAI_MODEL_NAME_CHAT` is used only when you pass `--correct`, which asks Sapat
to clean up the transcript with a chat model after transcription.

### Groq Cloud

Groq is useful when speed matters. Its Whisper-compatible endpoint can be very
fast for short feedback loops:

```bash
GROQCLOUD_API_KEY=your_groq_api_key
GROQCLOUD_MODEL=whisper-large-v3-turbo
GROQCLOUD_API_ENDPOINT=https://api.groq.com/openai/v1/audio/transcriptions
GROQCLOUD_MODEL_NAME_CHAT=llama3-8b-8192
```

The issue request specifically asks for more supported APIs. Groq is a good
example because Sapat exposes it through the same command pattern as OpenAI.

### Azure OpenAI

Azure OpenAI is a good fit for teams that already manage AI access through Azure
resources:

```bash
AZURE_OPENAI_API_KEY=your_azure_openai_key
AZURE_OPENAI_ENDPOINT=https://DEPLOYMENTENDPOINTNAME.openai.azure.com
AZURE_OPENAI_DEPLOYMENT_NAME_WHISPER=whisper
AZURE_OPENAI_API_VERSION_WHISPER=2024-06-01
AZURE_OPENAI_DEPLOYMENT_NAME_CHAT=gpt-4o
AZURE_OPENAI_API_VERSION_CHAT=2023-03-15-preview
```

Make sure `AZURE_OPENAI_DEPLOYMENT_NAME_WHISPER` matches the deployment name in
your Azure OpenAI resource. It is not always the same as the model name.

## Step 3: Run Your First Transcription

Sapat's CLI accepts a file or a directory as the input path. It requires an API
choice with `--api`, and it defaults to English with `--language en`.

Run an OpenAI transcription:

```bash
python -m src.sapat.script samples/demo-recording.mp4 --api openai --language en
```

Run the same file with Groq:

```bash
python -m src.sapat.script samples/demo-recording.mp4 --api groq --language en
```

Run it with Azure OpenAI:

```bash
python -m src.sapat.script samples/demo-recording.mp4 --api azure --language en
```

The command first converts the media file to MP3, then sends the audio to the
selected provider. Sapat writes transcript output next to the processed media,
which makes it easy to keep each transcript close to its source recording inside
the workspace.

If you have a folder of recordings, pass the folder path instead of a single
file:

```bash
python -m src.sapat.script samples --api groq --language en
```

That pattern is useful for processing a batch of customer interviews or a set of
short demo clips.

## Step 4: Tune Accuracy and Cost

The CLI exposes a few important options. These are the ones you will use most in
real projects.

### Specify the Language

If you know the language, set it explicitly. That reduces ambiguity and can make
transcription faster:

```bash
python -m src.sapat.script samples/demo-recording.mp4 --api openai --language de
```

### Add a Prompt

Use `--prompt` when the recording contains names, product terms, acronyms, or
technical vocabulary:

```bash
python -m src.sapat.script samples/demo-recording.mp4 \
  --api openai \
  --language en \
  --prompt "Daytona workspaces, dev containers, Sapat, Whisper, Groq Cloud"
```

A focused prompt is especially helpful for developer content, where a generic
speech model may confuse project names or command-line flags.

### Adjust Temperature

Sapat defaults to `--temperature 0.3`. Lower values are better for deterministic
technical transcripts:

```bash
python -m src.sapat.script samples/demo-recording.mp4 \
  --api groq \
  --language en \
  --temperature 0.1
```

### Choose Conversion Quality

The `--quality` option controls the MP3 conversion quality. Sapat accepts `L`,
`M`, and `H`, with `M` as the default:

```bash
python -m src.sapat.script samples/demo-recording.mp4 --api openai --quality H
```

Use `H` for recordings with background noise or multiple speakers. Use `M` for
most normal demos and meetings. Use `L` only when file size matters more than
accuracy.

### Correct the Transcript with a Chat Model

Pass `--correct` when you configured the provider's chat model variable and want
Sapat to clean up spelling, punctuation, and product names:

```bash
python -m src.sapat.script samples/demo-recording.mp4 \
  --api openai \
  --language en \
  --prompt "Daytona, Sapat, OpenAI Whisper, Groq Cloud" \
  --correct
```

This is useful for publishing workflows because the raw transcript is often
readable but not polished enough for documentation, subtitles, or blog drafts.

## Step 5: Verify the Output

After the command finishes, inspect the generated transcript file:

```bash
ls -lah samples
sed -n '1,80p' samples/demo-recording.txt
```

Check three things before you trust the result:

1. **Completeness:** the transcript should cover the whole recording, not only
   the first segment.
2. **Terminology:** product names, commands, and API names should be spelled
   correctly.
3. **Speaker context:** if the recording includes multiple people, review
   sections where interruptions or crosstalk occur.

For important material, run the same file through two providers and compare the
results:

```bash
python -m src.sapat.script samples/demo-recording.mp4 --api openai --language en
python -m src.sapat.script samples/demo-recording.mp4 --api groq --language en
```

OpenAI may be the safer default, Groq may be faster for iteration, and Azure may
fit enterprise governance requirements. Daytona makes this comparison easy
because the environment and files remain consistent while only the `--api` flag
changes.

## Common Issues and Troubleshooting

**Problem:** `ffmpeg: command not found`.

**Solution:** Install `ffmpeg` in the workspace image or with the package manager
before running Sapat.

**Problem:** The provider returns an authentication error.

**Solution:** Reopen `.env`, confirm the key name matches the provider, and make
sure you are running commands from the Sapat directory so `load_dotenv(".env")`
can find the file.

**Problem:** Azure returns a deployment or API-version error.

**Solution:** Confirm the deployment name and API version in the Azure Portal.
Use `AZURE_OPENAI_DEPLOYMENT_NAME_WHISPER` for the Whisper deployment and
`AZURE_OPENAI_API_VERSION_WHISPER` for the audio endpoint version.

**Problem:** The transcript misses product names or command syntax.

**Solution:** Add a concise `--prompt` with the vocabulary you expect to hear,
then rerun the command. For publishable output, also try `--correct`.

**Problem:** Large media files fail or time out.

**Solution:** Split long recordings into smaller files, use `--quality M` or
`--quality L` during testing, and process a directory in batches.

## Conclusion

Sapat gives you a simple way to test multiple AI transcription providers without
rewriting your workflow for each one. In a Daytona workspace, that workflow is
repeatable: clone the tool, install dependencies, configure `.env`, run the same
CLI with `--api openai`, `--api groq`, or `--api azure`, and review the transcript
next to the original media file.

For engineering teams, this setup is useful beyond one-off transcription. You
can turn demos into searchable notes, convert interviews into research material,
prepare captions for tutorials, or generate clean meeting records for handoff.
Start with a short recording, verify the output, then scale the same Daytona
workspace pattern to the rest of your audio and video library.
