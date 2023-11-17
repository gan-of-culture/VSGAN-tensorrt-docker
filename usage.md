# Getting started

## Pull the correct branch

If your CPU does not support `AVX512` then you need to use the branch `no_avx` since the master branch will not work for you. To check if your CPU does support `AVX512` you can either check your CPU flags by running:

```
grep avx512 /proc/cpuinfo
```

or run the inference.py and if you get a error `Illegal instruction` then your CPU does **not** support `AVX512`.

### no_avx

If you are using the `no_avx` branch you need to update the image in the file `compose.yaml` to `chin39/vsgan_tensorrt:latest`.

## Setting up inference.py

You need to update the filename in `inference.py` to match the name of the file you are trying to upscale.

## Creating a engine

If you have not yet created a engine or are encountering the error `vapoursynth.Error: operator(): open engine failed` then you need to create the engine for your GPU. All available models can be found [here](https://github.com/styler00dollar/VSGAN-tensorrt-docker/releases/tag/models).

To understand the difference between the models they have be named including a couple of their properties.

`[model name][level of denoising][scale of upscaling][opset][fixes for any issues]`

The `opset` stands for the different ONNX versions. Use a ff16 model if you don't have lots of VRAM since that model can be used with less resources.

Run the command as explained [here](https://github.com/styler00dollar/VSGAN-tensorrt-docker#vs-mlrt-c-trt) to create the engine. Make sure to update the `engine_path` to your newly created engine in the `inference_config.py` file if needed.

**Warnings**: 
- Make sure you add the argument --fp16 when creating a fp16 engine
- If you use the FP16 onnx you need to use `RGBH` colorspace, if you use FP32 onnx you need to use `RGBS` colorspace in `inference_config.py` 
- Engines are system specific, don't use across multiple systems
- Don't use reuse engines for different GPUs.
- If you run out of memory, then you need to adjust the resolutions in that command. If your video is bigger than what you can input in the command, use tiling.

If you encounter the following error:

```
1: [defaultAllocator.cpp::allocate::19] Error Code 1: Cuda Runtime (out of memory)
Requested amount of GPU memory (3887181312 bytes) could not be allocated. There may not be enough free memory for allocation to succeed.
2: [executionContext.cpp::ExecutionContext::470] Error Code 2: OutOfMemory (no further information)
Segmentation fault
```

you need to decrease the resolution set in the `max_shapes` argument and run it again. For example you could try `1x3x720x1280`.

## Running it

One way to run it is:

```vspipe -c y4m inference.py - | ffmpeg -i pipe: example.mkv -y```

many more ways are listed [here](https://github.com/styler00dollar/VSGAN-tensorrt-docker#usage).