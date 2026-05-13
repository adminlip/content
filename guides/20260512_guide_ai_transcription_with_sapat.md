---
title: 'AI Transcription with Sapat in Daytona'
description:
  'Learn how to use Sapat for AI-powered audio transcription using OpenAI, Groq, and Azure OpenAI APIs in Daytona workspaces.'
date: 2026-05-12
author: 'AI Agent'
tags: ['ai', 'transcription', 'daytona', 'sapat', 'openai', 'groq']
---

# AI Transcription with Sapat in Daytona

# Introduction

This guide will walk you through setting up and using [Sapat](https://github.com/nkkko/sapat), a powerful AI-powered transcription tool that supports multiple transcription APIs including OpenAI Whisper, Groq Cloud, and Azure OpenAI.

By the end of this guide, you'll be able to transcribe audio files in your [Daytona workspace](/definitions/20240819_definition_daytona%20workspace.md) using any of these APIs.

Sapat provides a unified interface for audio transcription, making it easy to switch between different AI providers based on your needs for speed, accuracy, or cost optimization.

## TL;DR

- Install Sapat in your Daytona workspace
- Configure API keys for OpenAI, Groq, or Azure OpenAI
- Run transcription with a single command
- Choose the best API for your use case

## Prerequisites

To follow this guide, you'll need:

- A [Daytona workspace](/definitions/20240819_definition_daytona%20workspace.md) set up
- An audio file you want to transcribe
- API keys for at least one of the supported services:
  - [OpenAI API](https://platform.openai.com/) key
  - [Groq Cloud API](https://console.groq.com/) key
  - [Azure OpenAI API](https://azure.microsoft.com/services/cognitive-services/openai/) credentials

## Step 1: Set Up Sapat in Daytona

### Step 1.1: Clone the Sapat Repository

First, clone the Sapat repository into your Daytona workspace:

```bash
cd ~/ && git clone https://github.com/nkkko/sapat.git
```

### Step 1.2: Install Dependencies

Navigate to the Sapat directory and install the required Python packages:

```bash
cd sapat
pip install -e .
```

### Step 1.3: Set Up Environment Variables

Copy the example environment file and configure your API keys:

```bash
cp .env.example .env
```

Edit the `.env` file with your API credentials. For OpenAI:

```
OPENAI_API_KEY=your_openai_api_key_here
OPENAI_MODEL=whisper-1
```

For Groq:

```
GROQCLOUD_API_KEY=your_groq_api_key_here
GROQCLOUD_MODEL=whisper
```

For Azure OpenAI:

```
AZURE_OPENAI_API_KEY=your_azure_key
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=your-deployment-name
```

## Step 2: Run Transcription

### Step 2.1: Transcribe with OpenAI

To transcribe an audio file using OpenAI Whisper:

```bash
sapat /path/to/audio.mp3 --api openai --language en
```

Or using the Python module:

```bash
python -m sapat /path/to/audio.mp3 --api openai --language en
```

### Step 2.2: Transcribe with Groq

To transcribe using Groq Cloud (faster inference):

```bash
sapat /path/to/audio.mp3 --api groq --language en --quality M
```

Or using the Python module:

```bash
python -m sapat /path/to/audio.mp3 --api groq --language en --quality M
```

### Step 2.3: Transcribe with Azure OpenAI

To transcribe using Azure OpenAI:

```bash
sapat /path/to/audio.mp3 --api azure --language en
```

Or using the Python module:

```bash
python -m sapat /path/to/audio.mp3 --api azure --language en
```

## Step 3: Advanced Options

### Step 3.1: Specify Language

By default, Sapat detects language automatically. You can specify explicitly:

```bash
python -m sapat /path/to/audio.mp3 --api groq --language de
```

### Step 3.2: Use Custom Prompts

Add context to improve transcription accuracy with a custom prompt:

```bash
python -m sapat /path/to/audio.mp3 --api openai --prompt "Technical discussion about machine learning"
```

### Step 3.3: Adjust Temperature

Control the creativity of the transcription:

```bash
python -m sapat /path/to/audio.mp3 --api groq --temperature 0.2
```

## Step 4: Confirmation

To confirm your transcription is working correctly:

1. Check the terminal output for the transcribed text
2. Verify the output file was created (if using `--output` flag)
3. Review the transcription for accuracy

## Common Issues and Troubleshooting

**Problem:** "API key not found" error

**Solution:** Ensure your `.env` file is properly configured and located in the Sapat directory. The environment variables must be set before running the script.

**Problem:** "Audio file too large" error

**Solution:** Sapat has a 25MB limit for audio files. Split your audio into smaller chunks or use the Azure API which supports larger files.

**Problem:** "Unsupported API" error

**Solution:** Make sure you're using a supported API name: `openai`, `groq`, or `azure` (case-insensitive).

## Conclusion

Congratulations! You've successfully set up Sapat in your Daytona workspace and learned how to transcribe audio using multiple AI providers. Here's a quick comparison to help you choose the right API:

- **OpenAI**: Best for accuracy and reliability
- **Groq**: Fastest inference, ideal for quick prototyping
- **Azure OpenAI**: Best for enterprise deployments with compliance requirements

For more information, visit the [Sapat GitHub repository](https://github.com/nkkko/sapat).
