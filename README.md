# llama-utils

Daemon for managing [`llama.cpp`](https://github.com/ggml-org/llama.cpp) instances and models

Run `llama-utils --help` for list of actions and arguments.

## description

This utility is similar to [`ollama`](https://github.com/ollama/ollama) in terms of core functionality, but with different approach.
While `ollama` comes bundled with `llama.cpp`, this tool does not - it uses `llama.cpp` as external dependency, utilizing it's built-in tools and scripts to operate.

This approach allows you to easily use latest `llama.cpp` features without having to wait for the maintainers to update the dependency, and makes this tool extremely lightweight.

## functionality

- managing `llama-server` instances
- downloading models from huggingface
- quantizing downloaded models

### future functionality ideas

- custom web-ui
- web search
- RAG

## requirements

- **Python 3** - required to run model conversion scripts from `llama.cpp`

### `llama.cpp` installation

`llama-utils` does not come with built-in `llama.cpp`. To get `llama.cpp`, you have to either:

- download pre-built libraries from [`llama.cpp` repository](https://github.com/ggml-org/llama.cpp/releases) - this can be done automatically
- build `llama.cpp` manually

Considering the fact that `llama.cpp` supports a lot of different architectures and configurations, and only some are provided via [releases](https://github.com/ggml-org/llama.cpp/releases) (and even with those, some users still may prefer to build it manually), `llama-utils` aims to give the user maximum flexibility, while still making it ergonomic to use in typical scenario.

By default, `llama-utils` looks for the `llama.cpp` installation in following directories (and order):

- Path specified by `--llama-cpp-path` argument, if provided
- Path from `paths.llama-cpp-install-dir` key in config, if provided
- System's `PATH` variable

If `llama.cpp` is not found in any of those paths, `llama-utils` can download latest (or manually specified) release from Github and extract it to user-specified directory (saving it as `paths.llama-cpp-install-dir` in config).

## configuration

llama-utils configuration is stored in `~/.llama-utils.toml` by default.
Configuration file location can be overridden with `-c/--config` argument.
On first run, a default configuration file will be created, and you will be asked to edit it and optionally change some environment-specific settings (like paths to model directories).

```toml
[paths]
# Path to directory with models.
# Will be created, if doesn't exist.
models-dir = "~/.llama-utils/models"
# Path to python's virtualenv used for llama.cpp scripts.
# Will be created, if doesn't exist.
python-venv-dir = "~/.llama-utils/python-venv"
# Path to installation directory of llama.cpp.
# This is optional, if llama.cpp is installed system-wide.
# However, if set - it will take the precedence over system-wide installation.
llama-cpp-install-dir = "/usr/local/bin"

[daemon]
# Daemon's port.
port = 10809
# First port used by llama-server instances.
# If multiple servers are running, they get next ports, e.g. 10811, 10812, etc.
first-server-port = 10810
# Maximum amount of llama-server instances (loaded models) running at the same time.
server-instances-limit = 4
```

### models storage

Models (both raw weights and quants) are stored in the directory specified in configuration.
The directory structure of `paths.models-dir` is as following:

```shell
models_dir
|-- model_a
    |-- model.safetensors
    |-- tokenizer.json
    |-- tokenizer_config.json
|-- model_b
    |-- model.safetensors
    |-- tokenizer.json
    |-- tokenizer_config.json
    |-- .llama-quants
        |-- model_b.Q8_0.gguf
        |-- model_b.Q4_K_M.gguf
|-- model_c
    |-- .llama-quants
        |-- model_c.Q6_K.gguf
        |-- model_c.Q5_K_M.gguf
```

When requesting a download from huggingface, `llama-utils` tries to be reasonably smart and download only the original weights or specific quants, instead of pulling the whole repository. This behavior can be overridden via `--download-everything` or `--download-exclude`/`--download-include` options.

## building

If you want to build llama-utils from source, you need latest stable Rust toolchain.
Runtime dependencies are not required for building.
