---
description: |
  ROS2 Jazzy talker/listener 테스트용 GitHub Actions 워크플로우를
  PR로 제안합니다. 머지 후 수동 실행하면 실제 테스트가 수행됩니다.

on:
  workflow_dispatch:

permissions:
  contents: read
  issues: read

network: defaults

tools:
  bash: [":*"]
  github:
    lockdown: false

safe-outputs:
  create-issue:
    title-prefix: "[ros2-test] "
    labels: [ros2, test-result]
  create-pull-request:
    title-prefix: "[ros2-test] "
    labels: [ros2, test-result]
---

# ROS2 Talker/Listener Test Setup

Create a Pull Request that adds a standard GitHub Actions workflow for ROS2 Jazzy talker/listener testing. This container cannot install system packages, so the actual test must run on a standard `ubuntu-latest` runner via a separate workflow.

## Important Constraints

- Do NOT attempt to install ROS2 in this container.
- Do NOT request write permissions — use the safe-outputs mechanism to create a PR.
- The goal is to propose a workflow file via PR.

## Steps

### 1. Create the workflow file content

Write a standard GitHub Actions workflow YAML file named `ros2-test-runner.yml` with this specification:

- **name**: `ROS2 Talker/Listener Test`
- **on**: `workflow_dispatch` (manual trigger only)
- **runs-on**: `ubuntu-latest`
- **permissions**: `issues: write`
- **Steps**:
  1. **Install ROS2 Jazzy**:
     - `sudo apt-get update && sudo apt-get install -y curl gnupg lsb-release`
     - Add ROS2 GPG key from `https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc`
     - Add ROS2 apt repository for Ubuntu from `packages.ros.org`
     - `sudo apt-get update && sudo apt-get install -y ros-jazzy-demo-nodes-cpp`
  2. **Run talker/listener test**:
     - Source `/opt/ros/jazzy/setup.bash`
     - Start `ros2 run demo_nodes_cpp talker` in background, capture output to a file
     - Sleep 3 seconds
     - Run `timeout 5 ros2 run demo_nodes_cpp listener`, capture output to a file
     - Kill the talker process
     - Check if listener output contains "Hello World" → PASS, otherwise FAIL
  3. **Create GitHub issue with results**:
     - Use `gh issue create` with title prefix `[ros2-test]` and labels `ros2,test-result`
     - Include: test result (PASS/FAIL), talker output (first 20 lines), listener output (first 20 lines)
     - Use the GITHUB_TOKEN for authentication via `GH_TOKEN` env var

### 2. Create a Pull Request

Use the safe-outputs mechanism to create a PR that adds `.github/workflows/ros2-test-runner.yml` to the repository. The PR should:

- Title: `Add ROS2 talker/listener test workflow`
- Body: explain that this is a standard GitHub Actions workflow for testing ROS2 Jazzy talker/listener demo on ubuntu-latest, triggered manually via workflow_dispatch
- Target branch: `main`

### 3. Create a summary issue

Create an issue summarizing what was done:
- The PR was created to add the ROS2 test runner workflow
- After merging, run it manually from the Actions tab via "Run workflow"
- The workflow will install ROS2 Jazzy, run talker/listener, and report results as an issue
