<div align="center">

<!-- Banner: upload a picture named banner.png under Data › Readme pictures (one of the generated images works well), then: ![Text to image](pictures/banner.png) -->

# Text to image

**One prompt, as many different images as you want, made on your own machine by open models from Hugging Face.**

The worker downloads a text-to-image model from Hugging Face once and runs it on the machine's NVIDIA GPU, Tenstorrent card or CPU. It turns a prompt into any number of images (20 in the demo), and each one appears in the view as soon as it is made. Models go from small (0.7 billion parameters, runs anywhere) to large (12 billion, for cloud GPUs).

![type](https://img.shields.io/badge/type-Generative%20AI-blue) ![language](https://img.shields.io/badge/language-Python-3776AB) ![runs on](https://img.shields.io/badge/runs%20on-NVIDIA%20%7C%20Tenstorrent%20%7C%20CPU-6f42c1) ![models](https://img.shields.io/badge/models-Hugging%20Face-yellow) [![made with](https://img.shields.io/badge/made%20with-EsaSim-0b7285)](https://esacode.com/)

</div>

## Overview

A diffusion model starts from random noise and removes it step by step, guided by the prompt, until an image is left. The noise it starts from comes from a **seed**: each seed gives a different image of the same prompt, and the same seed gives the same image again. So one prompt gives as many variants as you ask for, and any one can be made again.

The model is downloaded from Hugging Face the first time, only the parts the pipeline needs, into the worker's cache on the machine, and read from there on later runs. Nothing is sent to an online service: the prompt and the images stay on your machine until the results are uploaded to EsaSim.

## Flows

- **1 · One prompt, any number of images**: Segmind Vega makes 20 images of a rover in a Martian canyon (seeds 1 to 20; set **Images** to any number). The view shows them in a grid as they are made, the seconds each took and a table of seeds and prompts.
- **2 · One prompt, ten styles**: the same scene in ten styles (sunrise, night under the Milky Way, dust storm, seen from orbit, watercolour, pencil sketch, retro illustration…), two seeds each, with **Variations** added to the prompt in turn.
- **3 · Small to large: four models**: the same prompt and seeds on Vega RT, Vega, SSD-1B and SDXL 1.0 (0.7B to 3.5B parameters): their images side by side, and a chart of the seconds per image of each.

## The models

| Model | Size | Steps | Licence | Where it runs |
|---|---|---|---|---|
| **Vega RT** (Segmind) | 0.7B | 4 | Apache-2.0 | any machine, also a CPU (the fastest) |
| **Vega** (Segmind) | 0.7B | 25 | Apache-2.0 | NVIDIA GPU, 8 GB |
| **SSD-1B** (Segmind) | 1.3B | 25 | Apache-2.0 | NVIDIA GPU, 8 GB |
| **SDXL 1.0** (Stability AI) | 3.5B | 30 | CreativeML Open RAIL++-M | NVIDIA GPU, 12 GB (8 GB works, its parts taking turns) |
| **FLUX.1 schnell** (Black Forest Labs) | 12B | 4 | Apache-2.0, gated | a cloud GPU with about 40 GB (24 GB works, slower); needs a Hugging Face token |

*auto* (the default) uses Vega on a GPU or a Tenstorrent card and Vega RT on a CPU, at 768 px there. A model larger than the GPU still runs: its parts take turns on the GPU, or come layer by layer (slower). Any other text-to-image model in the diffusers format can be named instead (Model: Other), from Hugging Face or a folder on the machine.

## Where it runs

- **NVIDIA GPU**: half precision, the fastest place for these models. **Images at once** makes several images in one pass on a large GPU (a cloud machine).
- **Tenstorrent** (Wormhole, Blackhole): through **TT-XLA**, PyTorch/XLA with Tenstorrent's plugin (pip package `pjrt-plugin-tt`), installed on the machine as TT-Forge's documentation says. The model computes in bfloat16; the first image compiles its denoiser for the chip, which takes minutes, and the next ones reuse it. Diffusion models on TT-XLA are recent: if the compiler cannot run an operation of a model yet, *auto* goes on with the GPU or the CPU and the log says why; choosing **Tenstorrent** as the accelerator stops the run instead, with the compiler's message.
- **CPU**: any computer, with Vega RT at 768 px; much slower than a GPU.

**Accelerator** chooses: *auto* takes a Tenstorrent card, else an NVIDIA GPU, else the CPU; naming one stops the block if the machine does not have it.

## What you need

- A machine with the EsaSim worker (Project › Compute).
- Disk space for the models, downloaded the first time: Vega 3.3 GB (Vega RT adds 0.24 GB), SSD-1B about 4.5 GB, SDXL 1.0 6.9 GB, FLUX.1 schnell 34 GB. Flow 3 downloads the first four (about 15 GB). The block checks the free space first.
- Internet access the first time.
- Python packages, installed by the worker the first time: `torch` (with CUDA on NVIDIA machines), `diffusers`, `transformers`, `accelerate`, `safetensors`, `peft`, `huggingface_hub`, `pillow`. On a Tenstorrent machine, TT-XLA as above.
- No account or key, except for FLUX.1 schnell: sign in on Hugging Face, accept its conditions on its page, make a token (Settings › Access tokens, read) and save it once in your EsaSim profile (**Profile › Keys**). Every Text to image block then uses it when you run a flow; a token pasted in a block (**A key for this block only**) comes first. With a token, downloads also get Hugging Face's higher rate limits.

## How to run it

1. Under **Compute**, check that a machine is available for this project.
2. Open a flow under **Blueprint**; change the **Prompt** and the number of **Images** of the Text to image block if you like.
3. Press **Run**. The first run downloads the model (the progress shows on the block); then the images come one after the other.
4. Open the **View** to watch them arrive.

## Settings worth knowing

- **Prompt**: what the image shows, its light and its style. **Negative prompt**: what it should not show (models with guidance).
- **Images**: how many, any number. **First seed**: image n uses seed + n − 1; 0 gives new images at every run.
- **Images at once**: several images per pass of the model (large GPUs); the images are the same as one at a time.
- **Variations** (one per line): added to the prompt in turn, to explore styles as well as seeds.
- **Model** and **Accelerator**: as above. **Steps**, **Guidance**, **Width** and **Height**: 0 keeps the model's own.
- **Hugging Face token**: empty uses the token in the profile of whoever runs the flow.

## What you get

- **Images**: `image_01_seed1.png` … (more digits past 99 images) in Results, shown in the view's grid as they are made. A click opens an image full size on top of the grid: the arrows (or the arrow keys, or a swipe) go through all of them, × (or Esc) comes back. The grid keeps the latest 200 while the run goes on; every image is in Results.
- **Images table**: for each image its seed, prompt, variation, steps, guidance, size, time, model, licence and device.
- **Summary**: one row per run (model, parameters, steps, size, device, images at once, seconds per image, loading time, licence); flow 3 joins those of four models into a chart.
- **generation.json**: the model, its licence, the settings and the total time, to make the same images again.

## Models and licences

- **Segmind Vega**, [huggingface.co/segmind/Segmind-Vega](https://huggingface.co/segmind/Segmind-Vega), and **Vega RT**, [huggingface.co/segmind/Segmind-VegaRT](https://huggingface.co/segmind/Segmind-VegaRT): Apache-2.0.
- **Segmind SSD-1B**, [huggingface.co/segmind/SSD-1B](https://huggingface.co/segmind/SSD-1B): Apache-2.0.
- **Stable Diffusion XL 1.0**, [huggingface.co/stabilityai/stable-diffusion-xl-base-1.0](https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0): CreativeML Open RAIL++-M, which allows use, commercial included, with use restrictions listed in the licence.
- **FLUX.1 [schnell]**, [huggingface.co/black-forest-labs/FLUX.1-schnell](https://huggingface.co/black-forest-labs/FLUX.1-schnell): Apache-2.0; access is gated (accept its conditions on Hugging Face).
- With another model, the block reads the licence from its model card, records it with every image, and says so in the log when it limits use (non-commercial, research only) or is not stated.
- The code of the block is EsaSim's.

## Notes

- The images are not filtered: what they show comes from the prompt. Use the project responsibly, and check the licence of any model you add before using its images.
- The first image of a run takes longer: the model is loaded and warmed up (on Tenstorrent, compiled). The Summary gives it apart.
- Same seed, same prompt, same settings, same model: the same image, on any machine with the same kind of device.

<!-- esasim: files of the project -->

---

An [EsaSim](https://esacode.com/) project of the workspace Gianluca De Novi's workspace (generative-ai).

### Blueprints in this repository

- [1 · One prompt, any number of images](blueprints/1-one-prompt-any-number-of-images.json) · 5 blocks · version 6
- [2 · One prompt, ten styles](blueprints/2-one-prompt-ten-styles.json) · 4 blocks
- [3 · Small to large: four models](blueprints/3-small-to-large-four-models.json) · 12 blocks

Keys pasted in blocks (secret settings) are not saved here. Datasets, results and the team stay in EsaSim.
