# Strata ORCA IQ3_XXS — Portable Build

A portable Strata ORCA build featuring the **Qwen3.8-Flash-Next-Uncensored IQ3_XXS** model.

The build supports **Vision** and **Tools** and provides an OpenAI-compatible API.

## System Requirements

- **Minimum 64 GB of system RAM.**
- NVIDIA GPU with **24 GB VRAM** recommended.
- Tested on **NVIDIA RTX 3090 (24 GB VRAM)**.
- Other RTX GPUs, such as the RTX 4090 and RTX 5090, should work, but have **not been tested**.
- Install the latest **NVIDIA Studio Driver**.
  - Game Ready drivers should also work, but this has not been tested.

Tested driver information:

- NVIDIA-SMI: `617.42`
- KMD Version: `617.42`
- CUDA UMD Version: `13.4`

## Configuration

Before launching the server, edit the appropriate configuration file:

- `strata-orca-iq3_xxs.json`
- `strata-orca-iq3_xxs-cache1024.json`

Choose the configuration corresponding to the `.bat` file you intend to use.

### 1. Set the Strata ORCA directory

Replace all occurrences of:

`X:/Strata-ORCA/...`

with the actual path to your Strata ORCA directory.

### 2. Set the model paths

Replace the example model paths with the paths to your downloaded ORCA (Uncensored) model shards:

`X:/StrataModels/models/IQ3_XXS_ORCA/Qwen3.8-Flash-Next-Uncensored-IQ3_XXS-00001-of-00002.gguf`

`X:/StrataModels/models/IQ3_XXS_ORCA/Qwen3.8-Flash-Next-Uncensored-IQ3_XXS-00002-of-00002.gguf`

Download the model from Hugging Face:

[Qwen3.8-Flash-Next-Uncensored-GGUF — IQ3_XXS](https://huggingface.co/orcarouter/Qwen3.8-Flash-Next-Uncensored-GGUF/tree/main)

**Important:** The other required models and components are already included in the repository. You do not need to download them separately.

## Launching

Run one of the following batch files:

- `Start.bat`
- `Start_cache1024.bat`

Use the one corresponding to the configuration file you edited.

## Connecting to the Server

Strata Server provides a simple built-in chat interface and an **OpenAI-compatible API**.

You can connect compatible applications, including:

- Strata Desktop
- Open WebUI
- Other applications supporting the OpenAI API format

The build supports **Vision** and **Tools**.

## Building from Source

For instructions on building the project yourself, see:

[`README_Build.md`](README_Build.md)

For additional information about GPU compatibility, see:

[`my notes Supported video.txt`](my%20notes%20Supported%20video.txt)

## Main Repository

The main project repository, documentation, updates, and support:

[Strata — GitHub](https://github.com/Niko1221/Strata/tree/main)

Please direct questions and bug reports to the original repository.

## Disclaimer

I am not a programmer or a professional software developer.

This build was assembled with the assistance of AI tools. It is provided as-is, without guarantees.

If something does not work correctly, you will need to investigate the issue yourself and consult the documentation in the original repository.

Good luck, and have fun!