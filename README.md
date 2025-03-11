# MMAudio: Sounds generation for silent videos

Containerized version of the MMAudio sound generator. It is based on the MMAudio project, and lets you run it your own GPU.

Tested on RTX 3060 12GB, RTX 3090 TI, L40 and H100. Currently, only NVIDIA CPU's are supported, as the code relies on CUDA for the processing. 

During first start-up the container will acquire the latest model and code from [Rex Cheng's repo](https://github.com/hkchengrex/MMAudio) and the latest tencent/HunyuanVideo model from [Huggingface](https://huggingface.co/hkchengrex/MMAudio).

## Disk size and startup time

The container requires considerable disk space for storage of the AI models. On my setup I observe 7GB for the docker image itself, plus 15GB for cached data. Building the cache will happen the first time when you start the container. That can easily take 20 minutes or more. After that any restart should be faster.

It may be advisable to store the cache outside of the conatiner, e.g. by mounting a volume to /workspace.

## Variables

HVGP_AUTO_UPDATE: Automatically updates the models and inference scripts to the latest version upon container start-up (default: 0).
 - 0: Don't update automatically. Use the scripts that are bundled.
 - 1: Update and use the latest features / models. But also accept that this may being breaking changes.

This container does not provide much configuration, as many other configuration parameters can be changed through the web interface.

### Fixing caching issues

As the container updates the models to the latest available version, there is no guarantee that the cached files from previous start-ups are compatible with updated versions. I haven't encountered any issue yet. Though, should you run into issues, just removing the cache folder will cause the startup script to rebuild the cache from scratch, and thereby fix any inconsistencies.

## Command reference

### Build the container

Building the container is straight forward. It will build the container, based on NVIDIA's CUDA development container, and add required Python dependencies for bootstrapping HunyuanVideoGP. 

```bash
docker build -t olilanz/ai-mmaudio .
```

### Running the container

On my setup I am using the following parameters: 

```bash
docker run -it --rm --name ai-mmaudio \
  --shm-size 24g --gpus all \
  -p 7862:7860 \
  -v /mnt/cache/appdata/ai-mmaudio:/workspace \
  -e AUTO_UPDATE=1 \
  olilanz/ai-mmaudio
```
Note that you need to have an NVIDIA GPU installed, including all dependencies for Docker.

### Environment reference

I am running on a computer with an AMD Ryzen 7 3700X, 128GB Ram, an RTX 3090 TI with 24GB VRAM. CPU and Ram are plentiful. It runs stable in that configuration. The web UI handles out-of-memory errors gracefully. In case this happens, you can easily tweak the settings to balance the quality/speed/VRAM-requirements.

A video for 5 seconds in 960x544 (16:9, 540p) takes me about 15 minutes to render - with 40 inference steps and other high quality settings. 

## Resources
* For the non-GPU-Poor: https://github.com/hkchengrex/MMAudio

