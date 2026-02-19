---
description: |
  ROS2 Jazzy talker/listener 테스트를 트리거하고
  결과를 모니터링하여 GitHub Issue로 보고합니다.

on:
  workflow_dispatch:

permissions:
  contents: read
  issues: read
  actions: read

network: defaults

tools:
  bash: [":*"]
  github:
    lockdown: false

safe-outputs:
  create-issue:
    title-prefix: "[ros2-test] "
    labels: [ros2, test-result]
---

# ROS2 Talker/Listener Test Orchestrator

You orchestrate the ROS2 talker/listener test by triggering a standard GitHub Actions workflow that runs on a full Ubuntu runner.

## Important: Do NOT install ROS2 yourself

This container does NOT support system package installation (no root, no apt, read-only filesystem). Instead, trigger the `ros2-test-runner.yml` workflow which runs on `ubuntu-latest` with full system access.

## Steps

1. Trigger the ROS2 test runner workflow:
   - Run: `gh workflow run ros2-test-runner.yml --repo $GITHUB_REPOSITORY`
   - Wait a few seconds for the run to start

2. Monitor the workflow run:
   - Find the latest run: `gh run list --workflow=ros2-test-runner.yml --repo $GITHUB_REPOSITORY --limit 1 --json databaseId,status`
   - Poll until it completes: `gh run watch <run-id> --repo $GITHUB_REPOSITORY`

3. Check the result:
   - Get run status: `gh run view <run-id> --repo $GITHUB_REPOSITORY --json conclusion`
   - The runner workflow automatically creates an issue with test results

4. If the runner workflow already created an issue, summarize the result. If it failed before creating an issue, create one yourself with the failure details.
