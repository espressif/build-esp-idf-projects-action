# build-esp-idf-projects-action

This action builds multiple ESP-IDF projects in a single repository.

## Features

- Builds multiple ESP-IDF projects in a single repository
- Supports custom build configurations in `.idf_ci.toml` configuration file
- Supports building only related projects based on changed files in pull requests

## Environment Requirements

- The action must run in an ESP-IDF environment (e.g., using espressif/idf container)
- Python 3.7 or later is required
- ESP-IDF must be installed and accessible at the specified `idf_path` (default: `/opt/esp/idf`, configurable via `idf_path` input)

## Usage

### Basic Usage

To use this action in your workflow:

1. Create a workflow file (e.g., `.github/workflows/build.yml`)
2. Add the action to your workflow
3. Configure the inputs as needed

```yaml
name: Build ESP-IDF Projects
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    container: espressif/idf:latest
    steps:
      - uses: actions/checkout@v4
      - uses: build-esp-idf-projects-action@v1
        with:
          paths: "examples/hello_world"
```

### Inputs

| Input            | Description                                                | Default                         |
| ---------------- | ---------------------------------------------------------- | ------------------------------- |
| `paths`          | Semicolon-separated paths to the ESP-IDF projects to build | `.`                             |
| `target`         | Target to build for (e.g., esp32, esp32s2)                 | `all`                           |
| `parallel_index` | 1-based index of the parallel job when using matrix builds | `1`                             |
| `parallel_count` | Total number of parallel jobs when using matrix builds     | `1`                             |
| `idf_path`       | Path to the ESP-IDF installation                           | `/opt/esp/idf`                  |
| `modified_files` | (Internal) Semicolon-separated list of modified files      | _From PR changes_               |
| `artifact_name`  | (Internal) Name of the artifact to upload                  | `app_binaries_{parallel_index}` |

### Build Artifacts as Output

The action has no outputs, but it generates build artifacts that can be used in subsequent steps. The build artifacts contain:

- Bootloader binary (`bootloader.bin`)
- Partition table (`partition-table.bin`)
- Application binaries (`.bin` files)
- Build configuration (`sdkconfig.json`)
- Build logs (`build.log`)
- Size information (`size.json`)
- Flash arguments (`flasher_args.json`)

### Advanced Configuration

#### Project Selection

You can specify projects to build. All projects will be found recursively in the specified paths:

```yaml
- uses: build-esp-idf-projects-action@v1
  with:
    # single path
    paths: "examples"
```

Or semicolon-separated paths:

```yaml
- uses: build-esp-idf-projects-action@v1
  with:
    # multiple paths
    paths: "examples/hello_world;examples/getting_started"
```

#### Custom Build Configuration

The [`idf-ci`][idf-ci-project] profile system provides sensible defaults for building ESP-IDF projects. You can customize the build configuration by creating an `.idf_ci.toml` file in the root of your repository. For detailed information about profile configuration and options, see the [idf-ci documentation][idf-ci-profile].

#### Matrix Builds

For large repositories with many projects, you can use matrix builds to parallelize the build process:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    container: espressif/idf:latest
    strategy:
      matrix:
        chunk: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v4
      - uses: build-esp-idf-projects-action@v1
        with:
          paths: "examples"
          parallel_index: ${{ matrix.chunk }}
          parallel_count: 4
```

## Contributing

An external workflow is used to test the action in a real-world scenario: [`test-build.yml`](https://github.com/hfudev/build-and-test-esp-idf-projects-example/blob/master/.github/workflows/test-build.yml)

[idf-ci-project]: https://github.com/espressif/idf-ci
[idf-ci-profile]: https://espressif-idf-ci.readthedocs-hosted.com/en/latest/explanations/profiles.html#profiles
