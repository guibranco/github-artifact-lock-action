# GitHub Artifact Lock action

A reusable composite GitHub Action to simulate a **step-level mutex lock** using GitHub Artifacts as the locking mechanism. Useful for serializing access to shared resources (like test environments) across workflows or pull requests.

## 🚀 Features

- 🔒 Acquire a lock before a step (e.g., integration test, deployment).
- 🧼 Release the lock automatically.
- 🕒 Configurable wait/retry logic.
- ✅ Uses GitHub-native APIs only—no external services needed.

## 📦 Inputs

| Input         | Description                              | Default     |
|---------------|------------------------------------------|-------------|
| `lock-name`   | Name of the lock (artifact name)         | `test-lock` |
| `wait-seconds`| Wait time between attempts (in seconds)  | `10`        |
| `max-attempts`| Max number of retries before failing     | `30`        |

## 🧪 Example

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Acquire test lock 🔐
        uses: guibranco/github-artifact-lock-action@v1
        with:
          lock-name: test-lock

      - name: Run tests 🧪
        run: echo "Tests running safely..."

      - name: Release test lock 🔓
        if: always()
        uses: guibranco/github-artifact-lock-action/release-lock@v1
        with:
          lock-name: test-lock
```

## 🛠️ How It Works

1. Before a critical step, it checks the GitHub Artifacts list for an artifact named `lock-name`.
2. If found, it waits and retries.
3. Once free, it uploads a dummy artifact to "acquire" the lock.
4. After the step, the lock artifact is deleted to release it.

## 📄 License

MIT



