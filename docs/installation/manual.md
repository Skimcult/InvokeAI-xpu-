# Manual Install

!!! warning

    **Python experience is mandatory.**

    If you want to use Invoke locally, you should probably use the [launcher](./quick_start.md).

    If you want to contribute to Invoke or run the app on the latest dev branch, instead follow the [dev environment](../contributing/dev-environment.md) guide.

InvokeAI is distributed as a python package on PyPI, installable with `pip`. There are a few things that are handled by the launcher that you'll need to manage manually, described in this guide.

## Requirements

Before you start, go through the [installation requirements](./requirements.md).

## Walkthrough

We'll use [`uv`](https://github.com/astral-sh/uv) to install python and create a virtual environment, then install the `invokeai` package. `uv` is a modern, very fast alternative to `pip`.

The following commands vary depending on the version of Invoke being installed and the system onto which it is being installed.

1. Install `uv` as described in its [docs](https://docs.astral.sh/uv/getting-started/installation/#standalone-installer). We suggest using the standalone installer method.

    Run `uv --version` to confirm that `uv` is installed and working. After installation, you may need to restart your terminal to get access to `uv`.

2. Create a directory for your installation, typically in your home directory (e.g. `~/invokeai` or `$Home/invokeai`):

    === "Linux/macOS"

        ```bash
        mkdir ~/invokeai
        cd ~/invokeai
        ```

    === "Windows (PowerShell)"

        ```bash
        mkdir $Home/invokeai
        cd $Home/invokeai
        ```

3. Create a virtual environment in that directory:

    ```sh
    uv venv --relocatable --prompt invoke --python 3.12 --python-preference only-managed .venv
    ```

    This command creates a portable virtual environment at `.venv` complete with a portable python 3.12. It doesn't matter if your system has no python installed, or has a different version - `uv` will handle everything.

4. Activate the virtual environment:

    === "Linux/macOS"

        ```bash
        source .venv/bin/activate
        ```

    === "Windows (PowerShell)"

        ```ps
        .venv\Scripts\activate
        ```

5. Choose a version to install. Review the [GitHub releases page](https://github.com/invoke-ai/InvokeAI/releases).

6. Determine the package specifier to use when installing. This is a performance optimization.

    - If you have an Nvidia 20xx series GPU or older, use `invokeai[xformers]`.
    - If you have an Nvidia 30xx series GPU or newer, or do not have an Nvidia GPU, use `invokeai`.

7. Determine the torch backend to use for installation, if any. This is necessary to get the right version of torch installed. This is acheived by using [UV's built in torch support.](https://docs.astral.sh/uv/guides/integration/pytorch/#automatic-backend-selection)

    === "Invoke v5.12 and later"

        - If you are on Windows or Linux with an Nvidia GPU, use `--torch-backend=cu128`.
        - If you are on Linux with no GPU, use `--torch-backend=cpu`.
        - If you are on Linux with an AMD GPU, use `--torch-backend=rocm6.3`.
        - **In all other cases, do not use a torch backend.**

    === "Invoke v5.10.0 to v5.11.0"

        - If you are on Windows or Linux with an Nvidia GPU, use `--torch-backend=cu126`.
        - If you are on Linux with no GPU, use `--torch-backend=cpu`.
        - If you are on Linux with an AMD GPU, use `--torch-backend=rocm6.2.4`.
        - **In all other cases, do not use an index.**

    === "Invoke v5.0.0 to v5.9.1"

        - If you are on Windows with an Nvidia GPU, use `--torch-backend=cu124`.
        - If you are on Linux with no GPU, use `--torch-backend=cpu`.
        - If you are on Linux with an AMD GPU, use `--torch-backend=rocm6.1`.
        - **In all other cases, do not use an index.**

    === "Invoke v4"

        - If you are on Windows with an Nvidia GPU, use `--torch-backend=cu124`.
        - If you are on Linux with no GPU, use `--torch-backend=cpu`.
        - If you are on Linux with an AMD GPU, use `--torch-backend=rocm5.2`.
        - **In all other cases, do not use an index.**

8. Install the `invokeai` package. Substitute the package specifier and version.

    ```sh
    uv pip install <PACKAGE_SPECIFIER>==<VERSION> --python 3.12 --python-preference only-managed --force-reinstall
    ```

    If you determined you needed to use a torch backend in the previous step, you'll need to set the backend like this:

    ```sh
    uv pip install <PACKAGE_SPECIFIER>==<VERSION> --python 3.12 --python-preference only-managed --torch-backend=<VERSION> --force-reinstall
    ```

9. Deactivate and reactivate your venv so that the invokeai-specific commands become available in the environment:

    === "Linux/macOS"

        ```bash
        deactivate && source .venv/bin/activate
        ```

    === "Windows (PowerShell)"

        ```ps
        deactivate
        .venv\Scripts\activate
        ```

10. Run the application, specifying the directory you created earlier as the root directory:

    === "Linux/macOS"

        ```bash
        invokeai-web --root ~/invokeai
        ```

    === "Windows (PowerShell)"

        ```bash
        invokeai-web --root $Home/invokeai
        ```

## Intel XPU (Windows, experimental)

Intel XPU support is experimental and requires a working oneAPI + Level Zero setup. This workflow installs a
PyTorch XPU wheel and then installs InvokeAI from source without replacing torch. You will see dependency warnings
about the torch version - this is expected.

### Prerequisites

- Intel oneAPI Base Toolkit (2025.x recommended)
- Intel GPU drivers with Level Zero support
- Node.js (for building the Web UI)

### Steps (CMD)

1. Clone the repo and create a venv:

    ```bat
    git clone https://github.com/invoke-ai/InvokeAI.git
    cd InvokeAI
    python -m venv .venv
    .venv\Scripts\activate
    ```

2. Load oneAPI and verify the GPU is visible:

    ```bat
    call "C:\Program Files (x86)\Intel\oneAPI\2025.0\setvars.bat"
    sycl-ls
    ```

3. Install PyTorch XPU and verify it:

    ```bat
    pip install torch torchvision --index-url https://download.pytorch.org/whl/xpu
    python -c "import torch; print(torch.__version__); print(torch.xpu.is_available())"
    ```

    If no matching wheels are found, try the nightly index:

    ```bat
    pip install --pre torch torchvision --index-url https://download.pytorch.org/whl/nightly/xpu
    ```

4. Install InvokeAI from source without replacing torch, then pin numpy <2:

    ```bat
    pip install -e . --no-deps
    pip install "numpy<2" --force-reinstall --no-deps
    ```

5. Install the remaining dependencies (excluding torch/torchvision):

    ```bat
    pip install ^
      accelerate ^
      blake3 ^
      compel==2.1.1 ^
      Deprecated ^
      diffusers==0.36.0 ^
      dnspython ^
      dynamicprompts ^
      einops ^
      fastapi==0.118.3 ^
      fastapi-events ^
      gguf ^
      huggingface-hub ^
      mediapipe==0.10.14 ^
      onnx==1.16.1 ^
      onnxruntime==1.19.2 ^
      opencv-contrib-python ^
      picklescan ^
      prompt-toolkit ^
      pydantic ^
      pydantic-settings ^
      pypatchmatch ^
      python-multipart ^
      python-socketio ^
      PyWavelets ^
      requests ^
      safetensors ^
      semver~=3.0.1 ^
      sentencepiece==0.2.0 ^
      spandrel ^
      torchsde ^
      transformers>=4.56.0 ^
      uvicorn[standard]
    ```

    Note: `bitsandbytes` is optional on Windows and often fails to install.

6. Build the Web UI:

    ```bat
    npm install -g pnpm
    set PYTHONPATH=%CD%
    python scripts\generate_openapi_schema.py > invokeai\frontend\web\openapi.json
    cd invokeai\frontend\web
    pnpm typegen < openapi.json
    pnpm vite build
    cd ..\..\..
    ```

7. Create the config file and force XPU:

    ```bat
    notepad %USERPROFILE%\invokeai\invokeai.yaml
    ```

    Example config:

    ```yaml
    schema_version: 4.0.2
    device: xpu
    precision: float16
    attention_type: torch-sdp
    attention_slice_size: auto
    enable_partial_loading: false
    keep_ram_copy_of_weights: true
    device_working_mem_gb: 2
    force_tiled_decode: false
    sequential_guidance: false
    ```

8. Run InvokeAI:

    ```bat
    set INVOKEAI_DEVICE=xpu
    invokeai-web --root %USERPROFILE%\invokeai
    ```

## Headless Install and Launch Scripts

If you run Invoke on a headless server, you might want to install and run Invoke on the command line.

We do not plan to maintain scripts to do this moving forward, instead focusing our dev resources on the GUI [launcher](../installation/quick_start.md).

You can create your own scripts for this by copying the handful of commands in this guide. `uv`'s [`pip` interface docs](https://docs.astral.sh/uv/reference/cli/#uv-pip-install) may be useful.
