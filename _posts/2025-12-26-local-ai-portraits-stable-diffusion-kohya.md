---
title: "Fully Local AI Portraits with Stable Diffusion + kohya_ss (LoRA)"
description: "Step-by-step guide to train and generate ultra-realistic professional headshots locally with Stable Diffusion WebUI and kohya_ss LoRA."
categories: [AI, Computer Vision]
tags: [stable-diffusion, lora, kohya_ss, computer-vision, portraits, on-prem, ai]
---

You can produce LinkedIn- and CV-ready headshots entirely on your own machine—no cloud, no uploads. This post walks through the end-to-end local workflow: prerequisites, installs, data prep, LoRA training in kohya_ss, and generating professional portraits in AUTOMATIC1111.

## 1) System requirements
- **GPU:** NVIDIA RTX 3060/4060 or better, >= 8 GB VRAM  
- **RAM:** 16 GB recommended  
- **Disk:** SSD  
- **OS:** Windows 10/11 (64-bit)

## 2) Required software
- **Python 3.10.6 (exact version required)** — during install, check "Add Python to PATH".  
  Download: https://www.python.org/downloads/release/python-3106/  
- **Git** — https://git-scm.com/download/win

## 3) Stable Diffusion WebUI (AUTOMATIC1111)
Install:
<pre>
<code class="language-bash">
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
cd stable-diffusion-webui
</code></pre>
Run:
<pre><code class="language-bash">webui-user.bat
</code></pre>
UI: `http://127.0.0.1:7860`

## 4) kohya_ss for LoRA training
Install:
<pre>
<code class="language-bash">
git clone https://github.com/bmaltais/kohya_ss.git
cd kohya_ss
</code></pre>
Create venv and install deps:
<pre>
<code class="language-bash">
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
</code>
</pre>
Run:
<pre>
<code class="language-bash">
python kohya_gui.py
</code>
</pre>
UI: `http://127.0.0.1:7861`

## 5) Prepare training data
Folder layout:
<pre>
<code class="language-text">
training/
└── 12_personname man/
    ├── 001.jpg
    ├── 001.txt
    ├── 002.jpg
    ├── 002.txt
    └── ...
</code>
</pre>
Caption content for each `.txt`:
<pre>
<code class="language-text">
photo of personname man
</code>
</pre>
Tips: 15-20 clean, varied shots (angles, lighting, backgrounds) work well; avoid sunglasses/heavy filters.

## 6) LoRA training settings (good starting point)
- Base model: `runwayml/stable-diffusion-v1-5`
- Resolution: `512`
- Batch size: `2`
- Epochs: `12-15`
- UNet LR: `1e-4`
- Text Encoder LR: `5e-5`
- Rank: `16`
- Alpha: `16`
- Precision: `fp16`

## 7) Use the trained LoRA in WebUI
LoRA file lands at:
<pre>
<code class="language-text">
stable-diffusion-webui/models/Lora/personname_lora_v1.safetensors
</code>
</pre>

Add to your prompt:
<pre>
<code class="language-text">
<lora:personname_lora_v1:0.6>
</code>
</pre>

## 8) Professional portrait prompt (ready-to-use)
Main prompt:
<pre>
<code class="language-text">
professional portrait of a man in his late 30s,
mediterranean / turkish appearance,
natural male hairline with slightly receding temples,
slightly fuller cheeks,
soft jawline,
medium-density beard,
thick eyebrows,
neutral confident expression,
85mm lens,
camera distance around 1.4 meters,
upper torso visible,
dark blazer,
shirt without tie,
soft daylight lighting,
light neutral background,
ultra realistic,
high detail,
natural skin texture,
no beauty filter
</code>
</pre>

Negative prompt:
<pre>
<code class="language-text">
woman, female,
cross-eyed, asymmetrical eyes,
plastic skin,
beauty filter,
cartoon, anime
</code>
</pre>

## 9) Output formats
- LinkedIn: 1:1  
- CV: 4:5  
- Minimum resolution: 1024x1024  

## 10) Closing thoughts
With kohya_ss and Stable Diffusion running locally, you keep privacy, avoid cloud costs, and still get production-quality portraits. Tune prompts, adjust LoRA weight (`:0.4-0.8`), and iterate until you like the skin texture and lighting. Happy training!
