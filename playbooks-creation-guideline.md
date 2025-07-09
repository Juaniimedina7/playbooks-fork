# Satori CI Playbook Language Guide

This document outlines the structure and capabilities of the Satori CI playbook language, based on the analysis of the existing `.yml` files.

## Playbook Structure

Satori CI playbooks are YAML files that define a series of tasks to be executed in a specified environment. Each playbook consists of one or more of the following top-level sections:

- **`settings`**: (Required) Defines the overall configuration and metadata for the playbook.
- **`<tool_name>`**: (Required) A section named after the primary tool being used (e.g., `nmap`, `gitleaks`). It contains the core execution logic.
- **`install`**: (Optional) A top-level section for general installation steps.

---

## The `settings` Block

The `settings` block is a dictionary that contains all the configuration for the playbook. Its keys are detailed below.

### `name`
**(Required)** A string that provides a human-readable name for the playbook.

*Example:*
```yaml
name: "Nmap: Full Network Scan"
```

### `description`
**(Required)** A string, which can be multi-line, that describes the purpose and function of the playbook.

*Example:*
```yaml
description: |
  Nmap (Network Mapper) is an open-source tool for network exploration and security auditing.
  It allows network administrators to discover which devices are running on their network, find open ports and services, and detect vulnerabilities.
```

### `image`
**(Required)** The name of the Docker image to be used for the execution environment.

*Example:*
```yaml
image: debian
```

### `author`
**(Required)** A list of strings, typically URLs, crediting the author(s) of the tool or playbook.

*Example:*
```yaml
author:
  - https://github.com/nmap
```

### `gallery`
**(Required)** A list of strings, usually URLs to images or GIFs, that showcase the tool's functionality.

*Example:*
```yaml
gallery:
  - https://files.catbox.moe/wpfplf.png
```

### `example`
**(Required)** A string that demonstrates how to run the playbook using the `satori` CLI.

*Example:*
```yaml
example: satori run satori://scan/nmap.yml -d HOST="satori.ci" --report --output
```

### `files`
**(Optional)** A boolean. If set to `true`, it indicates that the playbook is designed to operate on local files in the working directory.

*Example:*
```yaml
files: True
```

### `cpu`
**(Optional)** An integer specifying the CPU resources to be allocated to the playbook's execution environment.

*Example:*
```yaml
cpu: 2048
```

### `memory`
**(Optional)** An integer specifying the memory resources (in MB) to be allocated to the execution environment.

*Example:*
```yaml
memory: 8192
```

### `timeout`
**(Optional)** An integer specifying the maximum execution time (in seconds) for the playbook. If not defined, a default value will be used.

*Example:*
```yaml
timeout: 900
```

### `redacted`
**(Optional)** A list of strings representing environment variable names whose values should be redacted from the logs.

*Example:*
```yaml
redacted:
  - AWS_ACCESS_KEY
  - AWS_SECRET_KEY
```

### `import`
**(Optional)** A list of strings specifying other playbooks to be imported. The format is `satori://<path_to_playbook>`.

*Example:*
```yaml
import:
  - satori://code/yamllint.yml
```

---

## The `<tool_name>` Block

This block is named after the playbook's primary tool (e.g., `nmap`, `bandit`, `trivy`).

### `install`
**(Optional)** A list of shell commands to install the tool.

*Example:*
```yaml
nmap:
  install:
    - apt-get update >>/dev/null; apt-get install -qy nmap >>/dev/null
```

### `help`
**(Optional)** A list of shell commands to display the tool's help message.

*Example:*
```yaml
nmap:
  help:
    - nmap --help
```

### `test`
**(Required)** A dictionary that defines the tests to verify the tool's functionality. It can contain assertions and one or more commands to execute.

- **`run`**: (Required within `test`) A list of shell commands that constitute the main test.
- **`assertReturnCode`**: (Optional) The expected return code of the `run` command.
- **`assertReturnCodeNot`**: (Optional) A return code that is not expected.
- **`assertStdoutContains`**: (Optional) A list of strings that must be present in the standard output.
- **`assertStdoutNotContains`**: (Optional) A list of strings that must not be present in the standard output.
- **`assertStderrContains`**: (Optional) A list of strings that must be present in the standard error.

*Simple Example:*
```yaml
trivy:
  test:
    assertStdoutNotContains:
      - HIGH
      - MEDIUM
    run:
      - trivy image ${{IMAGE}}
```

The `test` block can also contain sub-dictionaries to define named and more structured tests.

*Example with Sub-tests:*
```yaml
nmap:
  test:
    assertReturnCode: 0
    TCP:
      - nmap --top-ports 1000 -sS -A -sC --script vuln --open -oA tcp_scan ${{HOST}}
    UDP:
      - nmap -sU --top-ports 20 -A --open -oA udp_scan ${{HOST}}
```

---

## Variables

Playbooks can use variables to be more flexible and reusable. Variables are referenced with the `${{VARIABLE_NAME}}` syntax. These variables are defined at runtime via the `-d` or `--data` flag in the `satori` CLI.

*Example Usage:*
```yaml
run:
  - echo "Scanning host: ${{HOST}}"
  - trivy image ${{IMAGE_NAME}}
```
*Terminal Execution:*
```bash
satori run satori://container/trivy.yml -d IMAGE_NAME="python:3.4-alpine"
```
