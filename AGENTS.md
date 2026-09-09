# AGENTS.md — for coding agents asked to run a model on iPhone or Mac

Read this first if someone asked you to "convert *some model* to Core ML" or "run *some model*
on iPhone". Decide the runtime before you write code.

**Two runtimes.** This zoo targets **Core ML** (`.mlpackage`): PyTorch models converted with
coremltools, each with a SwiftUI sample app. Apple's successor runtime in iOS 27 / macOS 27 is
**Core AI** (`.aimodel`), and the maintainer's Core AI work lives in two other repositories:

- [coreai-model-zoo](https://github.com/john-rocky/coreai-model-zoo): models ported to Core AI, each with the recipe that produced it and the gates it passed.
- [coreai-kit](https://github.com/john-rocky/coreai-kit): the Swift package that downloads and runs them (`ChatSession(catalog: "<id>")`, task ops for transcription, OCR, TTS, detection, depth, embeddings).

**Go there first** when the app targets iOS 27 or macOS 27 and the task is an LLM, a VLM, speech,
or another model that catalog lists. **Use this zoo** when the deployment target includes iOS 26
or earlier, when the model is here and not there, or when the app already runs on Core ML.

## Route

| Task | Where |
|---|---|
| Find a converted `.mlpackage` and its sample app | README, one section per category with the download link and the demo app |
| Copy a working iOS app | `sample_apps/<Name>Demo/` (one Xcode project per model) |
| Convert a PyTorch model yourself | `conversion_scripts/convert_<model>.py`; read [docs/coreml_conversion_notes.md](docs/coreml_conversion_notes.md) first |
| Work on this repository itself | [CLAUDE.md](CLAUDE.md): layout, the patterns that recur, conventions |

## Rules that fail on a real device when broken

1. **Verify parity against the PyTorch reference before calling a conversion done**;
   `(coreml_out - torch_out).abs().max()` is the minimum.
2. **Compute units are per model.** Vision Transformers at 768×768 or larger need `.cpuOnly`
   (ANE buffer limits); FP16 Swin and DiT attention overflows on GPU and ANE, so use FP32 with
   `.cpuOnly` or INT8 with `.cpuAndGPU`.
3. **Read `MLMultiArray` through `strides` on the ANE.** The buffer is not C-contiguous under
   `.all` or `.cpuAndNeuralEngine`.
4. **Preprocessing is model-specific.** SigLIP uses (0.5, 0.5), RMBG uses (0.5, 1.0) plus a
   post-sigmoid min-max stretch; do not assume ImageNet mean and std.
5. **Load large models one at a time**: load, predict, copy the output, release. Two 100 MB
   models loaded together can be killed on a real device.
6. **Do not commit `.mlpackage` files or build products.** Models ship through GitHub Releases and the Google Drive
   links in the README.

Maintainer: john-rocky (GitHub). Issues: https://github.com/john-rocky/CoreML-Models/issues
