<div align="center">

<h1> ResAdapter: Domain Consistent Resolution Adapter for Diffusion Models  </h1>

Jiaxiang Cheng,
Pan Xie*,
Xin Xia,
Jiashi Li,
Jie Wu,
Yuxi Ren,
Huixia Li,
Xuefeng Xiao,
Min Zheng,
Lean Fu
(*Corresponding author)

AutoML, ByteDance Inc.

If our work is helpful for your research and application, please give us a star ✨. Our target is `1k+` :).
We will continue to maintain the repository for the betterment of our users. 

<a href='https://res-adapter.github.io/'><img src='https://img.shields.io/badge/Project-Page-green'></a> 
<a href='https://arxiv.org/abs/2403.02084'><img src='https://img.shields.io/badge/ Paper-Arxiv-red'></a> 
<a href='https://huggingface.co/jiaxiangc/res-adapter'><img src='https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Model-blue'></a>
![GitHub Org's stars](https://img.shields.io/github/stars/bytedance%2Fres-adapter)

[![Replicate](https://img.shields.io/badge/Demo-Replicate.com-purple)](https://replicate.com/bytedance/res-adapter) 
[![Hugging Face](https://img.shields.io/badge/Demo-%F0%9F%A4%97%20Hugging%20Face-66cdaa)](https://huggingface.co/spaces/ameerazam08/Res-Adapter-GPU-Demo)
[![ComfyUI](https://img.shields.io/badge/ComfyUI-ResAdapter-red)](https://github.com/jiaxiangc/ComfyUI-ResAdapter)


<strong> We propose ResAdapter, a plug-and-play resolution adapter for enabling diffusion models of arbitrary style domains to generate resolution-free images: no additional training, no additional inference and no style transfer.</strong>

<img src="assets/misc/dreamlike1.png" width="49.9%"><img src="assets/misc/dreamlike2.png" width="50%">
The above images are generated by ResAdapter with [dreamlike-diffusion-1.0](https://civitai.com/models/1274/dreamlike-diffusion-10)

</div>


## 🚀 Release
- `[2024/03/14]` We have completed [ComfyUI-ResAdapter](https://github.com/jiaxiangc/ComfyUI-ResAdapter), which will be open source in several days.
- `[2024/03/12]` We support [gradio demo with SDXL-Lightning]([https://replicate.com/bytedance/res-adapter](https://huggingface.co/spaces/ameerazam08/Res-Adapter-GPU-Demo)) in `huggingface.com` (by [@Ameer Azam](https://github.com/AMEERAZAM08))
- `[2024/03/12]` We support [gradio demo with SDXL-Lightning](https://replicate.com/bytedance/res-adapter) in `replicate.com` (by [@chenxiwh](https://github.com/chenxwh))
- `[2024/03/11]` We release the [resadapter-sdv15](https://huggingface.co/jiaxiangc/res-adapter/tree/main/sd1.5) that supports `128~1024` resolution.
- `[2024/03/05]` We release the paper about [resadapter](https://arxiv.org/abs/2403.02084) to arxiv.
- `[2024/03/04]` We release the inference codes and [resadapter-sdv15-i](https://huggingface.co/jiaxiangc/res-adapter/tree/main/sd1.5-i) that supports `128~512`, and [resadapter-xl-i](https://huggingface.co/jiaxiangc/res-adapter/tree/main/sdxl-i) that supports `256~1024`.


## 🔥 Standalone Example with [SDXL-Lightning](https://huggingface.co/ByteDance/SDXL-Lightning)

<div align=center> <strong>[ResAdapter]</strong> The following <strong>512x512</strong> images are generated by ResAdapter with SDXL-Lightning-Step4 </div>
<img src="assets/misc/sdxl-lighting-resadapter_512.png" width="100%">
<div align=center> <strong>[Baseline]</strong> The following <strong>512x512</strong> images are generated by SDXL-Lightning-Step4 </div>
<img src="assets/misc/sdxl-lighting_512.png" width="100%">
<div align=center> Prompt: A girl smiling. </div>

```python
# pip install diffusers, transformers, accelerate, safetensors, huggingface_hub
import torch
from diffusers import StableDiffusionXLPipeline, UNet2DConditionModel, EulerDiscreteScheduler
from huggingface_hub import hf_hub_download
from safetensors.torch import load_file
from torchvision.utils import save_image

generator = torch.manual_seed(12638721376)
width, height = 512, 512

base = "stabilityai/stable-diffusion-xl-base-1.0"
repo = "ByteDance/SDXL-Lightning"
ckpt = "sdxl_lightning_4step_unet.safetensors"  # Use the correct ckpt for your step setting!

# Load SDXL-Lighting.
unet = UNet2DConditionModel.from_config(base, subfolder="unet").to("cuda", torch.float16)
unet.load_state_dict(load_file(hf_hub_download(repo, ckpt), device="cuda"))
pipe = StableDiffusionXLPipeline.from_pretrained(base, unet=unet, torch_dtype=torch.float16, variant="fp16").to("cuda")
pipe.scheduler = EulerDiscreteScheduler.from_config(pipe.scheduler.config, timestep_spacing="trailing")

# Inference SDXL-Lighting
images = pipe(
    "A girl smiling",
    width=width,
    height=height,
    num_inference_steps=4,
    guidance_scale=0,
    num_images_per_prompt=5,
    output_type="pt",
    generator=generator,
).images
save_image(images, f"/baseline_{width}.png", normalize=True, padding=0)

# Load ResAdapter
pipe.load_lora_weights(
    hf_hub_download(
        repo_id="jiaxiangc/res-adapter",
        subfolder="sdxl-i",
        filename="resolution_lora.safetensors",
    ),
    adapter_name="res_adapter",
)
pipe.set_adapters(["res_adapter"], adapter_weights=[1.0])

# Inference ResAdapter
images = pipe(
    "A girl smiling",
    width=width,
    height=height,
    num_inference_steps=4,
    guidance_scale=0,
    num_images_per_prompt=5,
    output_type="pt",
    generator=generator,
).images
save_image(images, f"resadapter_{width}.png", normalize=True, padding=0)
```


## 🎉 Support ResAdapter in ComfyUI

🔥 [Update at 2024/03/14] We are pleased to announce that [ComfyUI-ResAdapter](https://github.com/jiaxiangc/ComfyUI-ResAdapter) is complete and users can install it to load the weights of ResAdapter.

🧊 However, due to the internal arrangement of our company, we will postpone open-sourcing the code of ComfyUI-ResAdapter for some time. We will open source it as soon as possible, please pay attention to us for the first time to obtain codes.

<img src="assets/misc/resadapter-1024x1024.png" width="100%">


## 🎉 Support ResAdapter Demos

🔥 [Update at 2024/03/12] We are happy to provide two demos in `replicate.com` and `huggingface.com`.

Relicate demo: [bytedance/res-adapter](https://replicate.com/bytedance/res-adapter), by ([@chenxiwh](https://github.com/chenxwh))

Huggingface demo: [ameerazam08/Res-Adapter-GPU-Demo](https://huggingface.co/spaces/ameerazam08/Res-Adapter-GPU-Demo), (by [@Ameer Azam](https://github.com/AMEERAZAM08))

<div align=center>
<img src="assets/misc/replicate_demo.jpeg" width="56.4%"><img src="assets/misc/huggingface_demo.jpeg" width="43.6%">

</div>




## ⏬ Installation

```bash
# Step1: Enter to res-adapter directory
cd res-adapter

# Step2: Install dependency
pip install -r requirements.txt

# Step3: Download diffusion models, and make the directory structure as follows:
models
├── res_adapter
│   ├── res_adapter-v1.5
│   ├── res_adapter-xl
├── diffusion_models
│   ├── ...
├── controlnet
│   ├── ...
├── ip_adapter
│   ├── ...
└── lcm-lora
    └──  ...
```

## ⏬ Download Models

### 1️⃣ Download ResAdapter

🎉 [Update at 2024/03/12] We release resadapter-sd1.5 that supports `128~1024`. 

|Models  | Parameters | Resolution Range | Links |
| --- | --- |--- | --- |
|resadapter-v1.5-i| 0.8M | 128<= x <= 512 | [Download](https://huggingface.co/jiaxiangc/res-adapter)|
|resadapter-xl-i| 0.4M | 256 <= x <= 1024 | [Download](https://huggingface.co/jiaxiangc/res-adapter)|
|resadapter-v1.5| 0.9M | 128 <= x <= 1024 | [Download](https://huggingface.co/jiaxiangc/res-adapter/tree/main/sd1.5)|
|resadapter-xl| 0.5M | 256 <= x <= 1536 | Coming soon|
<!-- |SDv1.5 | 860M | --- |[Download](https://huggingface.co/runwayml/stable-diffusion-v1-5)|
|SDXL1.0 |2.6B | --- |[Download](https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0) |  -->


<!-- |Dreamshaper|SDv1.5|2.5D | [Download](https://civitai.com/models/4384?modelVersionId=351306) -->




### 2️⃣ Download Diffusion Models

We provide some personalized models for sampling style images with ResAdapter.
More personalized models can be found in [CivitAI](https://civitai.com/).


|Models  | Structure Type |Domain Type |Links |
| --- | --- |--- |--- |
| **Base model**
|SDv1.5 | - | General |[Download](https://huggingface.co/runwayml/stable-diffusion-v1-5)|
|SDXL1.0 |- | General |[Download](https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0) |
| **Personalized model**
|RealisticVision|SDv1.5 |Realism | [Download](https://civitai.com/models/4201/realistic-vision-v60-b1)
|Dreamlike| SDv1.5 | Fantasy | [Download](https://civitai.com/models/1274/dreamlike-diffusion-10)
|DreamshaperXL|SDXL |2.5D | [Download](https://civitai.com/models/112902/dreamshaper-xl)
|AnimeartXL|SDXL |Anime | [Download](https://civitai.com/models/117259/anime-art-diffusion-xl)

### 3️⃣ Download Other Modules

We support demos about ControlNet, IP-Adapter, LCM-LoRA.

|Modules | Name | Type | Links |
| --- |--- | --- | --- |
|ControlNet| lllyasviel/sd-controlnet-canny |SD1.5 | [Download](https://huggingface.co/lllyasviel/sd-controlnet-canny)
|ControlNet| diffusers/controlnet-canny-sdxl-1.0 |SDXL | [Download](https://huggingface.co/diffusers/controlnet-canny-sdxl-1.0)
|IP-Adapter| h94/IP-Adapter | SD1.5/SDXL | [Download](https://huggingface.co/h94/IP-Adapter)
|LCM-LoRA| latent-consistency/lcm-lora-sdv1-5 |SD1.5 | [Download](https://huggingface.co/latent-consistency/lcm-lora-sdv1-5)
|LCM-LoRA| latent-consistency/lcm-lora-sdxl | SDXL| [Download](https://huggingface.co/latent-consistency/lcm-lora-sdxl)

## 🏠 ResAdapter in Inference Scripts

We provide simple scripts for sampling images of resadapter and baseline.

```bash
# Step1: Choose a task example of config file.
# Step2: Run the following script.
python main.py --config /path/to/file
```

### 0️⃣ Best Practice
😄 For better image generation, we provide two advice:
- For text to image tasks, please use personalized models instead of base models.
- For other tasks, use base models.

### 1️⃣ ResAdapter with Diffusion Models

<div align=center> <strong>[ResAdapter]</strong> The following <strong>960x1104</strong> images are generated by ResAdapter with <a href="https://civitai.com/models/4384?modelVersionId=109123">Dreamshaper-7</a>.</div>
<div align=center>
<img src="assets/misc/dreamshaper-1024/resadapter1.jpg" width="25%"><img src="assets/misc/dreamshaper-1024/resadapter2.jpg" width="25%"><img src="assets/misc/dreamshaper-1024/resadapter3.jpg" width="25%"><img src="assets/misc/dreamshaper-1024/resadapter4.jpg" width="25%">
</div>

<div align=center> <strong>[Baseline]</strong> The following <strong>960x1104</strong> images are generated by <a href="https://civitai.com/models/4384?modelVersionId=109123">Dreamshaper-7</a></div>
<div align=center>
<img src="assets/misc/dreamshaper-1024/baseline1.jpg" width="25%"><img src="assets/misc/dreamshaper-1024/baseline2.jpg" width="25%"><img src="assets/misc/dreamshaper-1024/baseline3.jpg" width="25%"><img src="assets/misc/dreamshaper-1024/baseline4.jpg" width="25%">
</div>

<div align=center> Text to image tasks. 

Prompt: (masterpiece), (extremely intricate), (realistic), portrait of a girl, the most beautiful in the world, (medieval armor), metal reflections, upper body, outdoors, intense sunlight, far away castle, professional photograph of a stunning woman detailed, sharp focus, dramatic, award winning, cinematic lighting, octane render unreal engine, volumetrics dtx, (film grain, blurry background, blurry foreground, bokeh, depth of field, sunset, motion blur), chainmail.</div>




### 2️⃣ ResAdapter with ControlNet

<div align=center> <strong>[ResAdapter]</strong> The following <strong>840x1246</strong> images are generated by ResAdapter with ControlNet(SDv15).</div>
<div align=center>
<img src="assets/misc/controlnet/condition_bird.jpg" width="20%"><img src="assets/misc/controlnet/bird_1_ResAdapter.jpg" width="20%"><img src="assets/misc/controlnet/bird_2_ResAdapter.jpg" width="20%"><img src="assets/misc/controlnet/bird_3_ResAdapter.jpg" width="20%"><img src="assets/misc/controlnet/bird_4_ResAdapter.jpg" width="20%">
</div>

<div align=center> <strong>[Baseline]</strong> The following <strong>840x1246</strong> images are generated by ControlNet(SDv15).</div>
<div align=center>
<img src="assets/misc/controlnet/condition_bird.jpg" width="20%"><img src="assets/misc/controlnet/bird_1_Baseline.jpg" width="20%"><img src="assets/misc/controlnet/bird_5_Baseline.jpg" width="20%"><img src="assets/misc/controlnet/bird_3_Baseline.jpg" width="20%"><img src="assets/misc/controlnet/bird_4_Baseline.jpg" width="20%">
</div>
<div align=center> Image to image tasks. 

Prompt: A bird.</div>



### 3️⃣ ResAdapter with ControlNet-XL

<div align=center> <strong>[ResAdapter]</strong> The following <strong>336x504</strong> images are generated by ResAdapter with ControlNet-XL(SDXL1.0).</div>
<div align=center>
<img src="assets/misc/controlnet-xl/condition_man.jpg" width="20%"><img src="assets/misc/controlnet-xl/man_0_ResAdapter.jpg" width="20%"><img src="assets/misc/controlnet-xl/man_1_ResAdapter.jpg" width="20%"><img src="assets/misc/controlnet-xl/man_2_ResAdapter.jpg" width="20%"><img src="assets/misc/controlnet-xl/man_3_ResAdapter.jpg" width="20%">
</div>

<div align=center> <strong>[Baseline]</strong> The above <strong>336x504</strong> images are generated by ControlNet-XL(SDXL1.0).</div>
<div align=center>
<img src="assets/misc/controlnet-xl/condition_man.jpg" width="20%"><img src="assets/misc/controlnet-xl/man_0_Baseline.jpg" width="20%"><img src="assets/misc/controlnet-xl/man_1_Baseline.jpg" width="20%"><img src="assets/misc/controlnet-xl/man_2_Baseline.jpg" width="20%"><img src="assets/misc/controlnet-xl/man_3_Baseline.jpg" width="20%">
</div>
<div align=center> Image to image tasks. The first column represents condition image. 

Prompt: A man.</div>

### 4️⃣ ResAdapter with IP-Adapter

<div align=center> <strong>[ResAdapter]</strong> The following <strong>864x1024</strong> images are generated by ResAdapter with IP-Adapter(SDv15).</div>
<div align=center>
<img src="assets/ip_adapter/ai_face2.png" width="20%"><img src="assets/misc/ip-adapter/resadapter3.jpg" width="20%"><img src="assets/misc/ip-adapter/resadapter4.jpg" width="20%"><img src="assets/misc/ip-adapter/resadapter5.jpg" width="20%"><img src="assets/misc/ip-adapter/resadapter7.jpg" width="20%">
</div>

<div align=center> <strong>[Baseline]</strong> The following <strong>864x1204</strong> images are generated by IP-Adapter(SDv15).</div>
<div align=center>
<img src="assets/ip_adapter/ai_face2.png" width="20%"><img src="assets/misc/ip-adapter/baseline3.jpg" width="20%"><img src="assets/misc/ip-adapter/baseline4.jpg" width="20%"><img src="assets/misc/ip-adapter/baseline5.jpg" width="20%"><img src="assets/misc/ip-adapter/baseline7.jpg" width="20%">
</div>
<div align=center> Face variation tasks. The first column represents face image.</div>


### 5️⃣ ResAdapter with LCM-LoRA

<div align=center> <strong>[ResAdapter]</strong> The following <strong>512x512</strong> images are generated by ResAdapter, LCM-XL-LoRA with <a href="https://huggingface.co/Lykon/dreamshaper-xl-1-0">Dreamshaper-XL.</a></div>
<div align=center>
<img src="assets/misc/lcm-lora/resadapter5.jpg" width="20%"><img src="assets/misc/lcm-lora/resadapter3.jpg" width="20%"><img src="assets/misc/lcm-lora/resadapter2.jpg" width="20%"><img src="assets/misc/lcm-lora/resadapter4.jpg" width="20%"><img src="assets/misc/lcm-lora/resadapter1.jpg" width="20%">
</div>

<div align=center> <strong>[Baseline]</strong> The following <strong>512x512</strong> images are generated by LCM-XL-LoRA with <a href="https://huggingface.co/Lykon/dreamshaper-xl-1-0">Dreamshaper-XL.</a></div>
<div align=center>
<img src="assets/misc/lcm-lora/baseline5.jpg" width="20%"><img src="assets/misc/lcm-lora/baseline3.jpg" width="20%"><img src="assets/misc/lcm-lora/baseline2.jpg" width="20%"><img src="assets/misc/lcm-lora/baseline4.jpg" width="20%"><img src="assets/misc/lcm-lora/baseline1.jpg" width="20%">
</div>

<div align=center> Text to image task. Step=4, CFG=1.5.

Prompt: portrait, action pose, slow motion, (old male human wizard) old male human wizard wearing yellow and black robes (majestic evoker cloth armor), (wrinkles, steampunk), (archmage robes, runic patterns), (insanely detailed, bloom), (analog), (high sharpness), (detailed pupils), (painting), (digital painting), detailed face and eyes, Masterpiece, best quality, (highly detailed photo), 8k, photorealistic, very long straight white and grey hair, grey streaks, ecstatic, (60-year old Austrian male), sharp, (older body), stocky, realistic, real shadow 3d, (highest quality), (concept art, 4k), (wizard labratory in backgound), by Michelangelo and Alessandro Casagrande and Greg Rutkowski and Sally Mann and jeremy mann and sandra chevrier and maciej kuciara, inspired by (arnold schwarzenegger) and (Dolph Lundgren) and (Albert Einstien)
</div>

<!-- **Best Practice**
- For interpolation (generating images below the training resolution), we recommend setting `adapter_alpha=1.0`. 
- For extrapolation (generating images above the training resolution), we recommend the following adapter_alpha settings: When the inference resolution is greater than 1.5x the training resolution, we recommend setting `0.2<adapter_alpha<0.6`. When the inference resolution is less than or equal to 1.5x the training resolution, we recommend setting `0.6< adapter_alpha<1.0`.
- We strongly recommend that you use the prompt corresponding to the personalized model, which helps to enhance the quality of the image. -->

## 🔨 TODO
- [x] Provide weights of resadapter-sdv15-i that supports `128~512`.
- [x] Provide weights of resadapter-sdv15-i that supports `128~1024`.
- [x] Provide weights of resadapter-xl-i that supports `256~1024`.
- [ ] Provide weights of resadapter-xl-i that supports `256~1536`.
- [x] Supporting resadapter in demos in huggingface.com.
- [x] Supporting resadapter in demos in replicate.com.
- [x] Supporting resadapter in **ComfyUI**.
- [ ] Supporting resadapter in **WebUI**.
<!-- - [ ] Supporting resadapter in **Huggingface** during mid to late March. -->
<!-- - [ ] Supporting resadapter in **Playground** during mid to late March, which is supported by. -->

## 🌟 Star History
[![Star History Chart](https://api.star-history.com/svg?repos=bytedance/res-adapter&type=Date)](https://star-history.com/#bytedance/res-adapter&Date)



## ✈️ Citation
```
@article{cheng2024resadapter,
  title={ResAdapter: Domain Consistent Resolution Adapter for Diffusion Models},
  author={Cheng, Jiaxiang and Xie, Pan and Xia, Xin and Li, Jiashi and Wu, Jie and Ren, Yuxi and Li, Huixia and Xiao, Xuefeng and Zheng, Min and Fu, Lean},
  booktitle={arXiv preprint arxiv:2403.02084},
  year={2024}
}
```

