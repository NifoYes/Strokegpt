# StrokeGPT — Full Guide (Zero → Hero)

This guide shows **exactly** how to run StrokeGPT locally, connect it to **LM Studio**, pick and configure the **language model**, pair **The Handy**, and (optionally) enable **ElevenLabs**. It includes the precise steps to start LM Studio from **Command Prompt** using `lms server start` (default **port 1234**).

---

## 1) Requirements

- **Windows 10/11** (Linux/macOS also work with similar commands)
- **Python 3.10+** (3.11 recommended)
- **LM Studio** installed (GUI; CLI `lms` available from LM Studio install)
- Internet connection for:
  - The Handy API
  - ElevenLabs (optional)
  - LLM download in LM Studio
- (Optional) **GPU** for faster inference

---

## 2) Install Python

Download from https://www.python.org/downloads/ and tick **“Add Python to PATH”** during setup.  
Verify:
```bash
python --version
```

---

## 3) Install LM Studio

Download and install from https://lmstudio.ai/

### 3.1 Download a model
Open LM Studio → **Explore** and download a model you want to use. Suggested:
- **Nous Hermes 2 Mistral 7B DPO** (quantization **Q4_K_S**): great quality/speed balance on a consumer PC.

Alternative (bigger/better but heavier):
- **Qwen2.5 14B Instruct** (requires more RAM/VRAM).

### 3.2 Start LM Studio API server **from Command Prompt**
Open **CMD** and run:
```bat
lms server start
```
This starts the LM Studio server on **port 1234** by default, exposing the OpenAI-compatible endpoint:
```
http://127.0.0.1:1234/v1/chat/completions
```
> If you prefer the GUI: LM Studio → **Developer → Local Server** → *Start Server*. Behavior is equivalent.

### 3.3 Quick test (optional)
PowerShell:
```powershell
curl -s -X POST http://127.0.0.1:1234/v1/chat/completions `
  -H "Content-Type: application/json" `
  -d "{ \"model\": \"nous-hermes-2-mistral-7b-dpo\", \"messages\": [{\"role\":\"user\",\"content\":\"Say hi\"}] }"
```
If you see a JSON with `"choices"`, the API is working.

> If you run the server on a different **port** or on a **remote IP**, note the new URL. You’ll set it in StrokeGPT.

---

## 4) Prepare StrokeGPT

Download/clone the StrokeGPT folder locally.  
Install Python dependencies from the project root:
```bash
pip install -r requirements.txt
```

Create your local config by copying the template:
```
my_settings.example.json  →  my_settings.json
```
Fill in the fields (locally only):
```json
{
  "handy_key": "YOUR_HANDY_API_KEY",
  "ai_name": "Erin the Succubus",
  "persona_desc": "Your character description and roleplay style.",
  "elevenlabs_api_key": "",
  "elevenlabs_voice_id": "",
  "min_speed": 10, "max_speed": 85,
  "min_depth": 5,  "max_depth": 95,
  "auto_min_time": 4, "auto_max_time": 10,
  "milking_min_time": 4, "milking_max_time": 10,
  "edging_min_time": 6, "edging_max_time": 14
}
```
- Get your **Handy key** from https://www.handyfeeling.com/app/ (login → Pair your Handy).  
- ElevenLabs fields are optional (only for TTS).

---

## 5) Configure the LLM connection (IMPORTANT)

StrokeGPT uses two variables in `app.py`:
```python
LLM_URL = "http://127.0.0.1:1234/v1/chat/completions"
MODEL_NAME = "nous-hermes-2-mistral-7b-dpo"
```
- **LLM_URL** → must match how you started LM Studio:
  - Local default (CLI `lms server start`): `http://127.0.0.1:1234/v1/chat/completions`
  - Remote LM Studio (example in LAN): `http://192.168.1.70:1234/v1/chat/completions`
- **MODEL_NAME** → must match the **exact model name** selected in LM Studio when serving.

**Golden rule:** whenever you change port/IP or model in LM Studio, update `LLM_URL` and/or `MODEL_NAME` in `app.py` accordingly.

---

## 6) Start StrokeGPT

1) Ensure **LM Studio** is already serving the model (CLI `lms server start` running).  
2) Launch the app:
```bash
python app.py
```
You should see:
```
🚀 Starting Handy AI app ...
🤖 LLM endpoint: http://127.0.0.1:1234/v1/chat/completions | model: nous-hermes-2-mistral-7b-dpo
 * Running on http://127.0.0.1:5000
```
Open your browser at **http://127.0.0.1:5000**.

---

## 7) How it works (Chat → Motion)

- Chat with the bot; it writes in **first person**, mixing **one short action beat** in *asterisks* with a **spoken line**.
- Text cues control The Handy:
  - “faster / speed up / più veloce” → increases **sp** (speed)
  - “go deeper / più profondo” → increases **dp** (depth center)
  - “longer strokes / colpi più lunghi” → increases **rng** (stroke range)
- Quick commands:
  - `stop` → emergency stop
  - `auto mode` / `you drive` / `take over` → **Auto Mode**
  - `start edging` / `tease and deny` → **Edging Mode**
  - `make me cum` / `finish me` → **Milking Mode**

### Phase Engine
- **WARM-UP**: low/medium pacing, teasing
- **ACTIVE**: medium/high, escalation within the same act
- **RECOVERY**: post-orgasm aftercare, slow down until you clearly opt back in

If not in Auto Mode, the app directly applies **sp/dp/rng** from the LLM’s response (clamped to your limits).

---

## 8) The Handy details

- Moves use percent scales (0–100):
  - **sp** = speed
  - **dp** = depth (center)
  - **rng** = stroke range (amplitude)
- Your `min_*`/`max_*` limits in `my_settings.json` enforce safe bounds.
- You can use manual controls in the UI at any time.

---

## 9) Media (optional)

Place your files here if you want the UI slideshow / ambient media:
```
static/updates/immagini   # .png .jpg .jpeg .gif .webp
static/updates/gif        # .gif (and images)
static/updates/video      # .mp4 .webm .mkv .mov .avi
static/updates/audio      # .mp3 .wav .ogg .m4a .flac
static/updates/botselfie  # images/videos for bot “selfie”
```

---

## 10) ElevenLabs (optional voice)

- Put `elevenlabs_api_key` and optional `elevenlabs_voice_id` in `my_settings.json`.
- You can enable/disable voice and switch voices from the UI.
- Audio is generated by `audio_service.py` and streamed to the browser.

---

## 11) Troubleshooting

- **UI works but no responses**  
  - LM Studio server not running or wrong URL/port.  
  - Test with the curl command in §3.3.
- **Timeout / connection refused**  
  - `lms server start` not running, wrong port, firewall blocking 1234.
- **LLM replies but device doesn’t move**  
  - Missing/invalid `handy_key`.  
  - Min/max limits too tight (no effective range).
- **Lag / slowness**  
  - Use a lighter model (7B Q4_K_S).  
  - In LM Studio, reduce **max tokens** or temperature.  

---

## 12) Project map (where things live)

- `app.py` → Flask server, UI routes, bridges to Handy/LLM/Audio  
- `llm_service.py` → sends `messages` to LM Studio (`/v1/chat/completions`) with `model=MODEL_NAME`  
- `handy_controller.py` → The Handy API (move/stop/nudge)  
- `audio_service.py` → ElevenLabs TTS (optional)  
- `background_modes.py` → Auto / Milking / Edging / Post-Orgasm logic  
- `settings_manager.py` → loads/saves `my_settings.json`  
- `index.html` → web UI

---

## 13) Quick start recap

1. **CMD**: `lms server start` → LM Studio API on **1234**  
2. `pip install -r requirements.txt`  
3. Create and fill `my_settings.json` (Handy key + optional TTS)  
4. Edit `LLM_URL` / `MODEL_NAME` in `app.py` if different from defaults  
5. `python app.py` → open http://127.0.0.1:5000

Have fun.
