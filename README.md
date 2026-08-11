# PhotonSimLink
An AI-powered assistant that translates plain English clinical requests into complex physics parameters to run native GPU laser simulations.

## What It Is
In medical device development, testing how lasers interact with human skin requires running massive, complex computer simulations called **Monte Carlo simulations**. Normally, these simulations require a specialized physics background to operate because you have to manually input highly specific math coefficients.
**This project bridges that gap.** It creates an AI-powered assistant that lets any engineer or clinician type a request in plain, conversational English (e.g., *"Simulate a laser scanning deep into dark skin"*), automatically translates it into the precise math parameters needed, and runs it on high-powered graphics cards (NVIDIA GPUs) in fractions of a second.

## How It Works (The 3-Step Pipeline)
* **1. Human-to-Math Translation:** The system takes your plain English text and maps it against a standardized rulebook (auto-generated `skill_params.md`) to figure out the exact numbers needed for skin absorption, light scattering, and laser positioning.
* **2. Dynamic VTS Physics Engine (`pVTS`):** Instead of static lookups, the system feeds those configurations into modular Python ports of VTS components to dynamically calculate exact optical properties from real chromophore spectral data.
* **3. Real Hardware Test & Report:** The validated numbers are fed straight into a 3D digital tissue grid on an NVIDIA graphics card. The system measures exactly where the light waves travel, catches the real calculation results, and hands them back to you in a clean, easy-to-read report.

Every response includes a status banner showing exactly what was used to generate it:
* 🟢 **FULLY VERIFIED** — real pVTS spectral data + real GPU hardware simulation
* 🟡 **PARTIAL** — real pVTS data, but GPU ran in CPU mock mode (no CUDA detected)
* 🟠 **PARTIAL** — real GPU hardware, but optical properties are a generic math placeholder (pVTS database not loaded)
* 🔴 **UNVERIFIED** — both optical properties and GPU simulation are placeholder/mock

## Key Problems This Solves
* **Rigorous Biophotonics Backend:** Powered by a modular `pVTS` Python implementation to dynamically calculate exact absorption and scattering coefficients per wavelength.
* **No More Serialization Crashes:** Fixed a glitch where the computer's raw matrix indices were clashing with Python's standard text converter, causing the application to crash.
* **Zero Boundary Wall Errors:** Patched a bug where launching the laser at the exact outer boundary wall caused the light simulation to instantly glitch out and return blank data (`nan` or Not a Number).
* **Accurate Beam Tracking:** Upgraded the tracking math to measure the laser light directly down its central core path instead of watering it down by averaging it with empty surrounding space.

## How to Run It (Deployment Options)
This pipeline is built to leverage parallel Monte Carlo computing power on any system utilizing an NVIDIA GPU. For this phase of the project, **Google Colab** is utilized as the primary, no-config sandbox for development and testing.

## Requirements
* **An NVIDIA GPU:** Required to run parallel photon-tracking simulation cores via CUDA.
* **A Groq API Key:** Required for fast LLM parameter translation via Llama 3.3 (available for free via the Groq Console).
* **`pVTS.zip`:** The physics backend package. Must be placed inside a `pVTS/` folder (i.e. the final path is `pVTS/pVTS.zip`) — `main.py` extracts it automatically on startup.
* **`resources/Spectra.txt`:** A spectral database file (chromophore extinction coefficients by wavelength). Required for pVTS to return real, verified optical properties — without it, the pipeline silently falls back to a generic math approximation (shown as 🟠/🔴 in the status banner).
* You do **not** need to manually create or upload `skill.md`/`skill_params.md` — `main.py` generates this itself on startup.

### Recommended: Google Colab Quickstart (Fastest Setup)
*Ideal for fast prototyping, testing, or internal research teams who want high-performance GPU access with zero local setup overhead.*

1. Launch a clean notebook in **Google Colab** and switch your hardware accelerator type to **T4 GPU** (*Runtime > Change runtime type > T4 GPU*).
2. Click the **Key icon** (Secrets panel) on the left sidebar, add a new secret named `GROQ_API_KEY`, paste your token, and enable notebook access.
3. Open the file drawer on the left side, create a `pVTS/` folder and upload `pVTS.zip` into it, then create a `resources/` folder and upload `Spectra.txt` into it.
4. Paste the complete pipeline Python script (`main.py`) into a code cell and run it — it installs its own dependencies (`groq`, `gradio`, `pmcx`, `numpy`, `matplotlib`) and generates its own `skill_params.md` automatically. Click the public `.gradio.live` tunnel link generated at the bottom of your console log to open your portal dashboard in any browser.

> **Files disappearing after a Colab restart or disconnect?** Colab's local storage is ephemeral — idle timeouts, session limits, or reconnecting can hand you a completely fresh VM with an empty disk. Mount Google Drive (`from google.colab import drive; drive.mount('/content/drive')`) and keep `pVTS.zip`/`Spectra.txt` there instead of re-uploading each time; copy them into the local `pVTS/`/`resources/` folders at the start of each run.

## Future Migration/Scalability Options
Because the codebase is built entirely on standard, modular Python packages, migrating this pipeline out of the Colab sandbox onto standalone hardware requires zero modifications to the underlying agent core logic.

### Option 1: Native Local Machine Workstation
*Ideal for engineers running on-premise hardware setups.*

1. **Clone the project files and install system packages:**
   ```bash
   git clone https://github.com/your-username/PhotonSimLink.git
   cd PhotonSimLink
   pip install -q -U groq gradio pmcx numpy matplotlib
   ```

2. **Save your Groq token directly to your operating system environment variables:**
   * **Linux/macOS:** `export GROQ_API_KEY="your-key-here"`
   * **Windows CMD:** `set GROQ_API_KEY="your-key-here"`

3. **Execute the native script loop:**
   ```bash
   python main.py
   ```

4. **Access the portal interface:**
   Open any web browser and navigate directly to the localhost address: `http://127.0.0.1:7860`

### Option 2: Dedicated Google Cloud Workstation (GCP Compute VM)
*Ideal for production scaling, 24/7 client portal web access, and standalone infrastructure hosting.*

1. **Launch a GPU VM:** Launch a **Compute Engine Instance** on GCP. Attach an `NVIDIA Tesla T4` or `L4` GPU and select a **Deep Learning VM Image (with CUDA pre-installed)** to automatically format your graphics architecture.

2. **Open Firewall Routing:** Navigate to your GCP Firewall configurations and create an opening rule setting targets to *all instances*, source IP ranges to `0.0.0.0/0`, and allow `tcp:7860`.

3. **Initialize and Serve:** SSH into your virtual machine, clone this repository, export your `GROQ_API_KEY`, and execute `python main.py`.

4. **Access the web interface:**
   Open any browser window and navigate directly to your cloud server's external address: `http://<YOUR_VM_EXTERNAL_IP>:7860`

## CPU Fallback/Development Mode
While actual simulation runs require a dedicated NVIDIA GPU for high-performance CUDA parallel processing, the codebase includes an automatic CPU fallback fixture. This is designed specifically for local development, testing, and migration when hardware accelerators aren't available. When active, the status banner will show 🟡 or 🔴 to make this visible rather than silently substituting mock data.

## Troubleshooting
* **`attempted relative import with no known parent package`:** Something is trying to load a `pVTS/*.py` file in isolation rather than as part of the `pVTS` package. Make sure `pVTS` is imported normally (`from pVTS.Tissue import ...`) rather than loaded file-by-file.
* **Status banner stuck on 🟠/🔴 despite `pVTS.zip` loading successfully:** Usually means `resources/Spectra.txt` wasn't found/loaded, or a file inside the extracted `pVTS/` package is an outdated copy missing a required fix. Re-run all cells top to bottom after any file changes — Python caches already-imported modules and won't pick up edits until re-imported.
* **Everything worked yesterday, nothing works today:** Almost always a Colab runtime restart wiped your local files. See the Google Drive tip above.

## Project Status & Roadmap
**PhotonSimLink** is currently under active development. The core architecture, 3D simulation pipeline, and Groq Llama 3.3 translation layer are currently being refined and tested.

- [x] Natural language prompt translation via Groq Llama 3.3
- [x] Safety validation layer (`skill_params.md` constraint checks)
- [x] Dynamic VTS physics backend (`pVTS`)
- [x] NVIDIA GPU execution pipeline (`pmcx.mcxlab()`)
- [x] Offline CPU mock mode and fallback fixture for hardware-free development
- [x] Status banner distinguishing verified vs. placeholder/mock results
- [ ] Conversational ambiguity handler *(Prompt users for missing details when generic requests lack target parameters) (Current Focus)*
- [ ] Final UI polish and automated regression testing
- [ ] Full source code release and stable tag
