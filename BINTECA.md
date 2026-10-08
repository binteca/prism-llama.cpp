# Binteca notes for this fork

This repository is Binteca's public fork of the
[PrismML llama.cpp fork](https://github.com/PrismML-Eng/llama.cpp), which is itself a fork of
[ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp). All credit for the inference
engine goes to the ggml / llama.cpp authors, and for the ternary kernels to PrismML.
The code is MIT-licensed; see [LICENSE](LICENSE), which is unchanged from upstream.

## Why we keep it

Stock and Homebrew llama.cpp cannot load the PQ2_0 and PTQ1_0 quantization types used by
the Ternary-Bonsai models. The PrismML fork adds the ternary kernels (Metal, CUDA and CPU)
and the Hadamard activation transform needed to run them. We use it to run
Ternary-Bonsai-2-27B PQ2_0 GGUFs on Apple Silicon.

## Branches, tags and releases

- `prism`: PrismML's upstream branch, kept as a plain mirror.
- `binteca` (default branch): upstream tag `prism-b10754-2459f68`
  (commit `2459f68b5c0eb26261fd5a81682004b93cd645ba`) plus this file. No code changes.
- Releases are tagged `binteca-<upstream tag>` and carry a macOS arm64 Metal build.

## Build (macOS arm64, Metal)

Portable release build, as used for the GitHub release:

```sh
cmake -B build-release -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DGGML_METAL=ON -DGGML_METAL_EMBED_LIBRARY=ON \
  -DGGML_NATIVE=OFF -DCMAKE_OSX_DEPLOYMENT_TARGET=13.3 \
  -DCMAKE_INSTALL_RPATH=@loader_path -DCMAKE_BUILD_WITH_INSTALL_RPATH=ON \
  -DLLAMA_OPENSSL=OFF -DGGML_RPC=ON \
  -DLLAMA_BUILD_TESTS=OFF -DLLAMA_BUILD_EXAMPLES=OFF \
  -DLLAMA_BUILD_TOOLS=ON -DLLAMA_BUILD_SERVER=ON \
  -DLLAMA_BUILD_UI=OFF -DLLAMA_USE_PREBUILT_UI=OFF
cmake --build build-release --config Release -j
```

Local build used for our first tests (tuned to the build machine):

```sh
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DGGML_METAL=ON \
  -DLLAMA_BUILD_TESTS=OFF -DLLAMA_BUILD_UI=OFF -DLLAMA_USE_PREBUILT_UI=OFF
cmake --build build --target llama-server llama-cli llama-bench
```

Notes:

- The web UI is not embedded (`LLAMA_BUILD_UI=OFF`), so use the OpenAI-compatible API or a
  separate client.
- The release build has no OpenSSL (`LLAMA_OPENSSL=OFF`), so HTTPS model downloads
  (`-hf`, `-mu https://...`) are not available. Download GGUFs separately and pass `-m`.

## Tested

- Model: Ternary-Bonsai-2-27B, PQ2_0 GGUF.
- Machine: MacBook Pro, Apple M5 Pro, 48 GB, macOS 27.
- Results with llama-server (Metal): about 26 tokens/s generation, about 400 tokens/s
  prompt processing, about 8.2 GiB peak memory.

## Known issues

- With Ternary-Bonsai, the "high" reasoning setting returns HTTP 500 from llama-server.
