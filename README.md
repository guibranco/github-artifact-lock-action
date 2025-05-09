# GitHub Artifact Lock Action

<div align="center">

[![GitHub repo](https://img.shields.io/badge/GitHub-guibranco%2Fgithub--artifact--lock--action-green.svg?style=flat-square&logo=github)](https://github.com/guibranco/github-artifact-lock-action)
[![GitHub last commit](https://img.shields.io/github/last-commit/guibranco/github-artifact-lock-action?color=green&logo=github&style=flat-square&label=Last%20commit)](https://github.com/guibranco/github-artifact-lock-action)
[![GitHub license](https://img.shields.io/github/license/guibranco/github-artifact-lock-action?color=green&logo=github&style=flat-square&label=License)](https://github.com/guibranco/github-artifact-lock-action)
![CI](https://github.com/guibranco/github-artifact-lock-action/actions/workflows/ci.yml/badge.svg)
[![wakatime](https://wakatime.com/badge/github/guibranco/github-artifact-lock-action.svg)](https://wakatime.com/badge/github/guibranco/github-artifact-lock-action)

</div>

## 📋 Overview

A reusable composite GitHub Action to implement a **step-level mutex lock** using GitHub Artifacts as the locking mechanism. This action enables serialized access to shared resources (like test environments, deployment targets, or external systems) across parallel workflows or pull requests.

## 🚀 Features

- 🔒 **Lock Acquisition**: Securely acquire locks before accessing shared resources
- 🔓 **Automatic Release**: Release locks reliably when operations complete
- ⏱️ **Configurable Timing**: Customize wait times and retry attempts
- 🌐 **Zero Dependencies**: Uses GitHub-native APIs only—no external services required
- 🛡️ **Failure Handling**: Ensures locks are released even when workflows fail
- 📊 **Workflow-Agnostic**: Works across different workflows and repositories

## 💡 When To Use

- Running integration tests that require exclusive access to test environments
- Managing deployments to shared staging environments
- Controlling access to external rate-limited APIs
- Preventing race conditions when multiple workflows update shared resources
- Implementing sequential processing when parallel execution isn't safe

## 📦 Installation

Add this action to your workflow YAML files. No additional setup required!

## ⚙️ Inputs

| Input | Description | Required | Default |
|-------|-------------|:--------:|---------|
| `lock-name` | Unique identifier for the lock (artifact name) | No | `test-lock` |
| `wait-seconds` | Wait time between retry attempts (in seconds) | No | `10` |
| `max-attempts` | Maximum number of retry attempts before failing | No | `30` |

## 📤 Outputs

This action doesn't provide explicit outputs, but creates and removes artifacts that serve as locks.

## 🧪 Usage Examples

### Basic Usage

```yaml
name: test
on: [push, pull_request]

permissions:
  actions: write
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Acquire test lock 🔐
        uses: guibranco/github-artifact-lock-action@v2.0.5
        with:
          lock-name: test-lock

      - name: Run tests 🧪
        run: echo "Tests running safely..."

      - name: Release test lock 🔓
        if: always()
        uses: guibranco/github-artifact-lock-action/release-lock@v2.0.5
        with:
          lock-name: test-lock
````

### Advanced Configuration

```yaml
name: deploy
on: [push]

permissions:
  actions: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Acquire deployment lock 🔐
        uses: guibranco/github-artifact-lock-action@v2.0.5
        with:
          lock-name: staging-environment-lock
          wait-seconds: 30
          max-attempts: 20

      - name: Deploy to staging 🚀
        run: ./deploy.sh --env staging

      - name: Release deployment lock 🔓
        if: always()
        uses: guibranco/github-artifact-lock-action/release-lock@v2.0.5
        with:
          lock-name: staging-environment-lock
```

### Multiple Locks

```yaml
name: integration
on: [push]

permissions:
  actions: write
  contents: read

jobs:
  integration:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Acquire database lock 🔐
        uses: guibranco/github-artifact-lock-action@v2.0.5
        with:
          lock-name: db-lock

      - name: Acquire API lock 🔐
        uses: guibranco/github-artifact-lock-action@v2.0.5
        with:
          lock-name: api-lock

      - name: Run integration tests 🧪
        run: npm run integration-tests

      - name: Release API lock 🔓
        if: always()
        uses: guibranco/github-artifact-lock-action/release-lock@v2.0.5
        with:
          lock-name: api-lock

      - name: Release database lock 🔓
        if: always()
        uses: guibranco/github-artifact-lock-action/release-lock@v2.0.5
        with:
          lock-name: db-lock
```

## 🛠️ How It Works

The action creates a locking mechanism by leveraging GitHub Artifacts storage:

1. **Check**: Before executing a critical step, it checks for an existing GitHub Artifact named `lock-name`
2. **Wait**: If the lock artifact exists, another workflow is using the resource, so this workflow waits
3. **Retry**: The action retries periodically until the lock is free or `max-attempts` is reached
4. **Acquire**: Once free, it uploads a small placeholder artifact with the `lock-name` to claim the lock
5. **Execute**: Your workflow steps run with exclusive access to the resource
6. **Release**: The release-lock action removes the artifact, freeing the resource for other workflows

## 🔄 Integration with Other Actions

This action works well with:

* Deployment actions like `actions/deploy-pages`
* Testing frameworks
* Database migration tools
* Any workflow requiring resource coordination

## 🐛 Troubleshooting

### Common Issues

* **Lock never releases**: Ensure the release step has `if: always()` to run even when prior steps fail
* **Timeouts**: Adjust `wait-seconds` and `max-attempts` for longer-running operations
* **Permissions**: Ensure workflow has sufficient permissions to read/write artifacts

### Debugging

Enable GitHub Actions debug logging by setting a secret named `ACTIONS_STEP_DEBUG` to `true`.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👏 Acknowledgments

* Inspired by the need for coordinating concurrent GitHub workflows
* Thanks to all contributors who helped improve this action
