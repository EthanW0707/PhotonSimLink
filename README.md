# PhotonSimLink
An AI-powered assistant that translates plain English clinical requests into complex physics parameters to run native GPU laser simulations.

## What It Is
In medical device development, testing how lasers interact with human skin requires running massive, complex computer simulations called **Monte Carlo simulations**. Normally, these simulations require a specialized physics background to operate because you have to manually input highly specific math coefficients. 
**This project bridges that gap.** It creates an AI-powered assistant that lets any engineer or clinician type a request in plain, conversational English (e.g., *"Simulate a laser scanning deep into dark skin"*), automatically translates it into the precise math parameters needed, and runs it on high-powered graphics cards (NVIDIA GPUs) in fractions of a second.

## How It Works (The 3-Step Pipeline)
* **1. Human-to-Math Translation:** The system takes your plain English text and maps it against a standardized rulebook (`skill.md`) to figure out the exact numbers needed for skin absorption, light scattering, and laser positioning.
* **2. Automatic Safety Check:** Before sending those numbers to the expensive computer processor, an automated Python safety layer checks the numbers. If the AI hallucinates or generates numbers that are physically impossible in biology, the system stops the test immediately to save computing power.
* **3. Real Hardware Test & Report:** The validated numbers are fed straight into a 3D digital tissue grid on an NVIDIA graphics card. The system measures exactly where the light waves travel, catches the real calculation results, and hands them back to you in a clean, easy-to-read report.

## Key Problems This Solves
* **No More Serialization Crashes:** Fixed a glitch where the computer's raw matrix indices were clashing with Python's standard text converter, causing the application to crash. 
* **Zero Boundary Wall Errors:** Patched a bug where launching the laser at the exact outer boundary wall caused the light simulation to instantly glitch out and return blank data (`nan` or Not a Number).
* **Accurate Beam Tracking:** Upgraded the tracking math to measure the laser light directly down its central core path instead of watering it down by averaging it with empty surrounding space.

## How to Run It (Deployment Options)

This pipeline is built to leverage parallel Monte Carlo computing power on any system utilizing an NVIDIA GPU. For this phase of the project, **Google Colab** is utilized as the primary, no-config sandbox for development and testing.

## Requirements
* **An NVIDIA GPU:** Required to run parallel photon-tracking simulation cores via CUDA.
* **A Groq API Key:** Required for fast LLM parameter translation via Llama 3.3 (available for free via the Groq Console).
* **The Constraints:** The `skill.md` file included in this repository must remain in the root directory for the AI parameters to validate correctly.

###  Recommended: Google Colab Quickstart (Fastest Setup)
*Ideal for fast prototyping, testing, or internal research teams who want high-performance GPU access with zero local setup overhead.*

1. Launch a clean notebook in **Google Colab** and switch your hardware accelerator type to **T4 GPU** (*Runtime > Change runtime type > T4 GPU*).
2. Install the containerized pipeline dependencies directly inside a terminal cell:
   ```bash
   pip install -q -U groq gradio pmcx numpy
   ```
3. Click the **Key icon** (Secrets panel) on the left sidebar, add a new secret named `GROQ_API_KEY`, paste your token, and enable notebook access.
4. Open the file drawer on the left side and upload your custom `skill.md` rulebook file directly into the main workspace folder.
5. Paste the complete pipeline Python script into your code cell and run it. Click the public `.gradio.live` tunnel link generated at the bottom of your console log to open your portal dashboard in any browser!

## Future Migration/Scalability Options
Because the codebase is built entirely on standard, modular Python packages, migrating this pipeline out of the Colab sandbox onto standalone hardware requires zero modifications to the underlying agent core logic.

### Option 1: Native Local Machine Workstation
*Ideal for engineers running on-premise hardware setups.*

1. **Clone the project files and install system packages:**
   ```bash
   git clone https://github.com
   cd PhotonSimLink
   pip install -q -U groq gradio pmcx numpy
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

3. **Initialize and Serve:** SSH into your virtual machine, clone this repository, export your`GROQ_API_KEY`, and execute `python main.py`.

4. **Access the web interface:**
   Open any browser window and navigate directly to your cloud server's external address: `http://<YOUR_VM_EXTERNAL_IP>:7860`


