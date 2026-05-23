# Forge Flash UI

A clean, minimal frontend for [Stable Diffusion WebUI Forge](https://github.com/lllyasviel/stable-diffusion-webui-forge) — inspired by Fooocus's streamlined workflow but with direct access to the controls that actually matter.
<img width="1454" height="854" alt="image" src="https://github.com/user-attachments/assets/ec44fb3f-597b-42a4-ba3b-dd77fb04e136" />


---

## Why

The default Gradio interface is powerful but noisy. Fooocus is clean but opinionated to a fault. This UI sits in between:

- **Everything unimportant is hardcoded** — sampler, scheduler, VAE, memory management. You never touch them unless you want to.
- **Everything important is front and center** — prompt, aspect ratio, and Hi-Res Fix scale are first-class controls on the main screen.
- **No Gradio dependency** — pure HTML/CSS/JS talking directly to Forge's REST API. Open it as a local file, host it anywhere, embed it in anything.

---

## Features

- Single-file — `forge_ui.html`, no build step, no Node, no Python
- Live model switching via `/sdapi/v1/options`
- Configurable backend host — works with local instances and remote servers (RunPod, etc.)
- Hi-Res Fix slider with auto-coupled Denoising Strength
- Aspect ratio presets optimized for SDXL pixel counts (1:1, 16:9, 9:16, 3:4, 4:3)
- Real-time progress bar + live preview during generation
- Seed display and lock after each generation
- Abort via `/sdapi/v1/interrupt`
- CUDA OOM error detection with human-readable messages
- PNG metadata retrieval
- Nordic Noir dark theme — JetBrains Mono + Syne

---

## Requirements

- [Stable Diffusion WebUI Forge](https://github.com/lllyasviel/stable-diffusion-webui-forge) installed and running
- A modern browser (Chrome, Firefox, Edge)
- No other dependencies

---

## Setup

### 1. Launch Forge with API enabled

Forge must be started with the `--api` flag and CORS open. Add these arguments to your launch command:

**Windows (`webui-user.bat`)**
```bat
set COMMANDLINE_ARGS=--listen --port 7860 --api --cors-allow-origins=* --no-half-vae
```

**Linux / macOS (`webui-user.sh`)**
```bash
export COMMANDLINE_ARGS="--listen --port 7860 --api --cors-allow-origins=* --no-half-vae"
```

**Or launch directly:**
```bash
python launch.py --listen --port 7860 --api --cors-allow-origins=* --no-half-vae
```

> `--no-half-vae` is recommended to avoid black image artifacts. Remove it if you are tight on VRAM.

### 2. Open the UI

Once Forge is running, open `forge_ui.html` directly in your browser:

```
File → Open File → forge_ui.html
```

Or serve it locally if you prefer (any static file server works):

```bash
# Python
python -m http.server 8080

# Node
npx serve .
```

Then navigate to `http://localhost:8080/forge_ui.html`.

### 3. Connect

The default host is `http://127.0.0.1:7860`. If your Forge instance runs on a different port or a remote machine, update the **Host** field in the top bar. The status indicator turns green when the connection succeeds and your model list loads automatically.

---

## Remote / Cloud Usage

If you are running Forge on a cloud GPU (RunPod, Vast.ai, etc.), replace the host field with your instance's public URL:

```
https://your-pod-id-7860.proxy.runpod.net
```

Make sure Forge is launched with `--cors-allow-origin=*` on the remote machine as well.

---

## Hardcoded Defaults

The following settings are intentionally fixed. Edit them directly in the `<script>` block in `forge_ui.html` if you want different defaults:

| Parameter | Value | Reason |
|---|---|---|
| `sampler_name` | `DPM++ 2M` | Best quality/speed balance for SDXL |
| `scheduler` | `Karras` | Consistent results across step counts |
| `hr_upscaler` | `Latent` | Fast, VRAM-efficient |
| `hr_second_pass_steps` | 50% of main steps | Balanced quality without doubling render time |
| `override_settings.sd_vae` | `Automatic` | Uses the model's bundled VAE |

---

## API Endpoints Used

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/sdapi/v1/progress` | Health check + live progress polling (500 ms) |
| `GET` | `/sdapi/v1/sd-models` | Populate model dropdown on load |
| `GET` | `/sdapi/v1/options` | Read active model checkpoint |
| `POST` | `/sdapi/v1/options` | Switch model |
| `POST` | `/sdapi/v1/txt2img` | Generate image |
| `POST` | `/sdapi/v1/interrupt` | Abort active generation |

Full Forge API reference: `http://127.0.0.1:7860/docs`

---

## VRAM Notes

Hi-Res Fix multiplies VRAM usage significantly. Rough guidance for a single generation at 1024×1024 base:

| Hr-scale | Effective resolution | VRAM (SDXL, fp16) |
|---|---|---|
| 1.0× (off) | 1024 × 1024 | ~6 GB |
| 1.5× | 1536 × 1536 | ~9 GB |
| 2.0× | 2048 × 2048 | ~14 GB |

If you hit a CUDA out-of-memory error, the UI will show a specific message. Lower the Hi-Res scale or the base resolution first.

---

## Roadmap

- [ ] img2img mode
- [ ] Style presets (prompt suffix injection)
- [ ] Session history with seed recall (`localStorage`)
- [ ] ControlNet panel
- [ ] Prompt token counter

---

## License

MIT — do whatever you want with it.
