# Archetect Render Action

![Latest Release](https://img.shields.io/github/v/release/archetect-actions/archetect-render?style=flat-square&label=Latest%20Release&color=blue)

A GitHub Action to render Archetypes using the [Archetect](https://github.com/archetect/archetect) tool. This composite action downloads the Archetect binary, processes JSON answers, and executes the render command to generate code from templates.

For complete Archetect documentation, visit [https://archetect.github.io](https://archetect.github.io).

## Description

This action provides a streamlined way to use Archetect within GitHub workflows for code generation. It supports:

- ✅ **Multi-platform support**: Linux x86_64, macOS x86_64, macOS ARM64
- ✅ **Flexible versioning**: Pin to specific Archetect versions or use latest
- ✅ **JSON answers integration**: Pass template variables as JSON objects
- ✅ **Robust error handling**: Validates inputs and provides clear error messages
- ✅ **Template flexibility**: Support for local directories, Git repositories, and remote sources

## Usage

### Basic Usage

```yaml
- name: Render Archetype
  uses: archetect-actions/archetect-render@v1
  with:
    source: "https://github.com/my-org/my-archetype.git"
    destination: "./generated"
    answers: |
      {
        "project_name": "my-project",
        "author": "John Doe",
        "license": "MIT"
      }
```

### Advanced Usage with Repository Dispatch

```yaml
- name: Render Archetype with Dynamic Answers
  uses: archetect-actions/archetect-render@v1
  with:
    version: "v2.0.4"
    source: "https://github.com/my-org/rust-service-archetype.git"
    destination: "./services/${{ github.event.client_payload.service_name }}"
    answers: ${{ toJson(github.event.client_payload.answers) }}
    args: "-vv"
```

## Inputs

| Input         | Description                                                                                            | Required | Default  |
| ------------- | ------------------------------------------------------------------------------------------------------ | -------- | -------- |
| `version`     | The Archetect version to use ([releases](https://github.com/archetect/archetect/releases))             | No       | `v2.0.4` |
| `source`      | The source directory or Git repo containing an Archetype                                               | **Yes**  |          |
| `destination` | The destination of the rendered output                                                                 | No       | `.`      |
| `answers`     | Answers in JSON Object (key/value) format. Can also be applied using `args`                            | No       | `{}`     |
| `args`        | Additional command-line arguments for the archetect command (e.g., `-a key=value`, `-s switch`, `-vv`) | No       | `""`     |

## Outputs

This action does not produce explicit outputs, but renders the archetype files to the specified destination directory.

## Examples

### Example 1: Simple Project Generation

```yaml
name: Generate New Service
on:
  workflow_dispatch:
    inputs:
      service_name:
        description: "Name of the service"
        required: true
      author:
        description: "Author name"
        required: true

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Generate Service
        uses: archetect-actions/archetect-render@v1
        with:
          source: "https://github.com/my-org/service-archetype.git"
          destination: "./services/${{ github.event.inputs.service_name }}"
          answers: |
            {
              "service_name": "${{ github.event.inputs.service_name }}",
              "author": "${{ github.event.inputs.author }}",
              "created_date": "${{ github.run_id }}"
            }
```

### Example 2: Repository Dispatch Integration

This example shows how to trigger archetype rendering from another repository:

**Dispatching Workflow (in source repository):**

```yaml
name: Release and Generate
on:
  push:
    tags:
      - "v*"

jobs:
  trigger-generation:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Extract version
        id: version
        run: echo "version=${GITHUB_REF#refs/tags/}" >> $GITHUB_OUTPUT

      - name: Trigger downstream generation
        uses: peter-evans/repository-dispatch@v3
        with:
          token: ${{ secrets.DISPATCH_TOKEN }}
          repository: my-org/target-repo
          event-type: generate-code
          client-payload: |
            {
              "source_version": "${{ steps.version.outputs.version }}",
              "answers": {
                "version": "${{ steps.version.outputs.version }}",
                "build_date": "${{ github.run_id }}",
                "git_sha": "${{ github.sha }}",
                "project_name": "my-generated-project"
              }
            }
```

**Receiving Workflow (in target repository):**

```yaml
name: Generate from Archetype
on:
  repository_dispatch:
    types: [generate-code]

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Render Archetype
        uses: archetect-actions/archetect-render@v1
        with:
          source: "https://github.com/my-org/my-archetype.git#${{ github.event.client_payload.source_version }}"
          destination: "./generated"
          answers: ${{ toJson(github.event.client_payload.answers) }}

      - name: Commit generated files
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add .
          git commit -m "Generate from archetype ${{ github.event.client_payload.source_version }}" || exit 0
          git push
```

### Example 3: Matrix Strategy for Multiple Outputs

```yaml
name: Generate Multiple Projects
on:
  workflow_dispatch:

jobs:
  generate:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        project:
          - name: "api-service"
            type: "rest-api"
            port: 8080
          - name: "worker-service"
            type: "worker"
            port: 8081

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Generate ${{ matrix.project.name }}
        uses: archetect-actions/archetect-render@v1
        with:
          source: "https://github.com/my-org/microservice-archetype.git"
          destination: "./services/${{ matrix.project.name }}"
          answers: |
            {
              "service_name": "${{ matrix.project.name }}",
              "service_type": "${{ matrix.project.type }}",
              "port": ${{ matrix.project.port }},
              "enable_metrics": true
            }
```

### Example 4: Local Archetype with Custom Args

```yaml
- name: Generate from Local Template
  uses: archetect-actions/archetect-render@v1
  with:
    source: "./templates/my-archetype"
    destination: "./output"
    answers: |
      {
        "component_name": "UserService",
        "namespace": "com.example.services"
      }
    args: "-vv -U"
```

## Platform Support

| Platform | Architecture          | Status           |
| -------- | --------------------- | ---------------- |
| Linux    | x86_64                | ✅ Supported     |
| macOS    | x86_64                | ✅ Supported     |
| macOS    | ARM64 (Apple Silicon) | ✅ Supported     |
| Windows  | x86_64                | ❌ Not supported |

## Error Handling

The action includes robust error handling:

- **Invalid JSON**: Automatically falls back to empty object `{}`
- **Download failures**: Clear error messages with download URLs
- **Platform detection**: Explicit error for unsupported platforms
- **Command validation**: Validates all inputs before execution

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Related Projects

- [Archetect](https://github.com/archetect/archetect) - The core Archetect tool
- [Archetect Actions](https://github.com/archetect-actions) - Collection of Archetect GitHub Actions
