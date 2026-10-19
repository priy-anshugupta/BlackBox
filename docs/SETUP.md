# Setup and local operation

Start with the platform-specific commands in [the README](../README.md). Run all commands from the repository root unless stated otherwise.

## Runtime and hardware

Python 3.11/3.12 is recommended for dependency compatibility. The project declares Python 3.10+; the migration also verifies against the existing Windows Python 3.13 environment. Node.js 22 matches the dashboard Docker build. No GPU is required for the sandbox or CPU diagnosis.

The pretrained semantic/NLI feature encoders can be downloaded from Hugging Face during the first initialization. Internet-free agent execution does not guarantee an internet-free first installation. Existing model caches can be reused; an unavailable encoder may cause the feature extractor to fall back rather than provide the trained semantic representation.

For GPU inference, install the PyTorch build appropriate to your driver before the other requirements. CPU installation is the simplest starting point. Avoid assuming that the recorded GPU latency applies to your computer.
