---
title: "Poetic Lens"
date: 2024-12-11
draft: false
author: "An Nguyen"
tags:
  - Python
  - PyTorch
  - Computer Vision
  - Generative AI
  - BLIP
  - GPT-2

image: 
description: ""
toc: true
mathjax: true
repoName: poetic-lens
---
## Overview

Poetic Lens is an AI-powered pipeline that transforms photographs into poetry by combining computer vision and text generation. Using a webcam, the system captures images, analyzes their content with the BLIP vision model, and generates original poems inspired by what it "sees."

<video controls autoplay loop muted playsinline width="100%">
  <source src="/videos/poem-cat.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

---

## AI Pipeline

1. **Capture a Photo** — Take a picture using your webcam.  
2. **Generate Caption** — The BLIP model describes the photo (e.g., “a photograph of a beautiful sunset over mountains”).  
3. **Create Prompt** — The caption is truncated into a short poetic prompt (“beautiful sunset over”).  
4. **Generate Poem** — The fine-tuned GPT-2 model uses the prompt as a seed to generate an original, multi-stanza poem.
5. **Output** — The final poem is saved to a text file and displayed in a pop-up window.

📸 **Camera** → 🧠 **BLIP Caption** → ✍️ **Prompt Extraction** → 🪄 **GPT-2 Poem Generation** → 📄 **poem.txt**

---

## Example

**Input:**  
A photo of a woman by a window.

**BLIP Caption:**  
> “a woman sitting by a window”

**Prompt Extracted:**  
> “woman sitting by”

**Generated Poem (excerpt):**  
> a woman sitting by  
> the whispers of rain’s reply  
> folds the silence into rhyme  
> as day dissolves in time  

---
## Customization

The system is designed with **modularity** in mind. It can be easily reconfigured or retrained to explore different creative styles:

- **Adjust prompt length:** Modify the truncation parameters in `blip.py`.
- **Control poem verbosity:** Change the `max_new_tokens` parameter in `gen_poem.py`.
- **Retrain or fine-tune:** Use different poetry datasets or fine-tune the GPT model on specific poetic styles.

---

## References

This project builds on Andrej Karpathy’s GPT and Salesforce’s BLIP.
