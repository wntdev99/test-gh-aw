---
description: |
  ROS2 노드 동작 종합 테스트. micromamba + RoboStack으로 ROS2를 설치하고
  Pub/Sub, Service, Action, Lifecycle, CLI 도구 등 다양한 기능을 검증하여
  결과를 GitHub Issue로 보고합니다.

on:
  workflow_dispatch:

permissions:
  contents: read
  issues: read

network:
  allowed:
    - defaults
    - "conda.anaconda.org"

tools:
  bash: [":*"]
  github:
    lockdown: false

safe-outputs:
  create-issue:
    title-prefix: "[ros2-test] "
    labels: [ros2, test-result]
---

# ROS2 Node Comprehensive Test

ROS2를 설치하고 다양한 노드 동작 테스트를 수행한 뒤, 전체 결과를 GitHub Issue로 보고합니다.

## Critical Environment Constraints

You are running inside a sandboxed chroot container with these hard limitations:

- **Non-root user** (`runner`, uid=1001) — `sudo` is NOT available
- **No apt/dpkg** — `/etc/apt/` and `/var/lib/apt/` directories do NOT exist
- **Read-only `/dev/shm`** — Python multiprocessing and system conda will fail
- **No Docker daemon** — `docker` CLI exists but daemon is not running
- **Network firewall** — only specific allowed domains are accessible
- **`gh` CLI is NOT authenticated** — do NOT use `gh` commands for GitHub operations. You MUST use the `create_issue` safe-output MCP tool to create issues.

**Allowed network domains include**: `github.com`, `raw.githubusercontent.com`, `conda.anaconda.org`

## Phase 1: ROS2 Environment Setup

### Step 1: Download micromamba

```bash
curl -fsSL https://github.com/mamba-org/micromamba-releases/releases/latest/download/micromamba-linux-64 -o /tmp/micromamba
chmod +x /tmp/micromamba
export MAMBA_ROOT_PREFIX="$HOME/micromamba"
mkdir -p "$MAMBA_ROOT_PREFIX"
```

Do NOT use `micro.mamba.pm` (blocked by firewall). Do NOT use system `conda` (fails on read-only `/dev/shm`).

### Step 2: Install ROS2 packages

```bash
/tmp/micromamba create -n ros_env \
  -c conda-forge \
  -c robostack-staging \
  --root-prefix "$MAMBA_ROOT_PREFIX" \
  --no-rc \
  -y \
  ros-humble-demo-nodes-cpp \
  ros-humble-demo-nodes-py \
  ros-humble-example-interfaces \
  ros-humble-lifecycle \
  ros-humble-ros2cli \
  ros-humble-ros2topic \
  ros-humble-ros2node \
  ros-humble-ros2service \
  ros-humble-ros2action \
  ros-humble-ros2param \
  ros-humble-ros2interface \
  ros-humble-ros2lifecycle
```

If some packages are not found, install what is available and skip the rest. At minimum `ros-humble-demo-nodes-cpp` and `ros-humble-demo-nodes-py` are required.

### Step 3: Activate environment

```bash
eval "$(/tmp/micromamba shell activate -s bash -n ros_env --root-prefix "$MAMBA_ROOT_PREFIX")"
ros2 --version
```

Record the ROS2 version for the report.

## Phase 2: Test Execution

Run each test below independently. For each test, record PASS/FAIL and capture output. If a test hangs, use `timeout` to prevent blocking. Always clean up background processes after each test.

### Test 1: Topic Pub/Sub (Talker/Listener)

The fundamental ROS2 communication test.

```bash
ros2 run demo_nodes_cpp talker > /tmp/test1_talker.txt 2>&1 &
PID=$!
sleep 3
timeout 5 ros2 run demo_nodes_cpp listener > /tmp/test1_listener.txt 2>&1 || true
kill $PID 2>/dev/null; wait $PID 2>/dev/null
```

- **PASS condition**: listener output contains "Hello World"

### Test 2: ROS2 CLI Introspection

Verify that ROS2 CLI tools can discover running nodes, topics, and interfaces.

```bash
# Start a talker node in background
ros2 run demo_nodes_cpp talker > /dev/null 2>&1 &
PID=$!
sleep 3

# Test CLI tools
ros2 node list > /tmp/test2_nodes.txt 2>&1
ros2 topic list > /tmp/test2_topics.txt 2>&1
ros2 topic info /chatter > /tmp/test2_topic_info.txt 2>&1
ros2 interface show std_msgs/msg/String > /tmp/test2_interface.txt 2>&1

kill $PID 2>/dev/null; wait $PID 2>/dev/null
```

- **PASS condition**: `ros2 node list` shows the talker node AND `ros2 topic list` includes `/chatter`

### Test 3: Service Server/Client

Test ROS2 service request-response pattern.

```bash
# Start add_two_ints server
ros2 run demo_nodes_cpp add_two_ints_server > /tmp/test3_server.txt 2>&1 &
PID=$!
sleep 3

# Call the service
timeout 10 ros2 service call /add_two_ints example_interfaces/srv/AddTwoInts "{a: 2, b: 3}" > /tmp/test3_client.txt 2>&1 || true

kill $PID 2>/dev/null; wait $PID 2>/dev/null
```

- **PASS condition**: client output contains the sum `5`

### Test 4: Parameter Operations

Test ROS2 parameter get/set on a running node.

```bash
ros2 run demo_nodes_cpp talker > /dev/null 2>&1 &
PID=$!
sleep 3

ros2 param list /talker > /tmp/test4_param_list.txt 2>&1
ros2 param get /talker use_sim_time > /tmp/test4_param_get.txt 2>&1

kill $PID 2>/dev/null; wait $PID 2>/dev/null
```

- **PASS condition**: `ros2 param list` returns parameters without error

### Test 5: Python Node (demo_nodes_py)

Verify Python-based ROS2 nodes also work.

```bash
ros2 run demo_nodes_py talker > /tmp/test5_talker.txt 2>&1 &
PID=$!
sleep 3
timeout 5 ros2 run demo_nodes_py listener > /tmp/test5_listener.txt 2>&1 || true
kill $PID 2>/dev/null; wait $PID 2>/dev/null
```

- **PASS condition**: listener output shows received messages

### Test 6: Multi-Node Topic Communication

Run multiple publishers on the same topic and verify a single subscriber receives from all.

```bash
ros2 run demo_nodes_cpp talker > /dev/null 2>&1 &
PID1=$!
ros2 run demo_nodes_py talker > /dev/null 2>&1 &
PID2=$!
sleep 3

timeout 5 ros2 topic echo /chatter std_msgs/msg/String > /tmp/test6_echo.txt 2>&1 || true

kill $PID1 $PID2 2>/dev/null; wait $PID1 $PID2 2>/dev/null
```

- **PASS condition**: `ros2 topic echo` captured messages from the topic

### Test 7: Topic Pub from CLI

Publish a message directly from CLI and verify reception.

```bash
ros2 run demo_nodes_cpp listener > /tmp/test7_listener.txt 2>&1 &
PID=$!
sleep 2

# Publish 3 messages from CLI
timeout 5 ros2 topic pub -t 3 /chatter std_msgs/msg/String "{data: 'CLI Test Message'}" > /tmp/test7_pub.txt 2>&1 || true
sleep 2

kill $PID 2>/dev/null; wait $PID 2>/dev/null
```

- **PASS condition**: listener received "CLI Test Message"

### Test 8: Topic Hz & Bandwidth Measurement

Measure the publishing frequency of a talker node.

```bash
ros2 run demo_nodes_cpp talker > /dev/null 2>&1 &
PID=$!
sleep 3

timeout 5 ros2 topic hz /chatter > /tmp/test8_hz.txt 2>&1 || true
timeout 5 ros2 topic bw /chatter > /tmp/test8_bw.txt 2>&1 || true

kill $PID 2>/dev/null; wait $PID 2>/dev/null
```

- **PASS condition**: `ros2 topic hz` reports an average rate (approximately 1 Hz for demo talker)

## Phase 3: Results Report

After all tests complete, use the `create_issue` safe-output MCP tool to create an issue. Do NOT use `gh` CLI.

**Issue title**: `Node Comprehensive Test - X/8 Passed`

**Issue body** should include the following sections formatted in Markdown:

### Summary Table

Create a table with columns: Test Name | Result | Details

| Test | Result | Details |
|------|--------|---------|
| 1. Topic Pub/Sub | PASS/FAIL | ... |
| 2. CLI Introspection | PASS/FAIL | ... |
| 3. Service Call | PASS/FAIL | ... |
| 4. Parameters | PASS/FAIL | ... |
| 5. Python Nodes | PASS/FAIL | ... |
| 6. Multi-Node Pub | PASS/FAIL | ... |
| 7. CLI Pub | PASS/FAIL | ... |
| 8. Topic Hz/BW | PASS/FAIL | ... |

### Environment Info

- ROS2 distro and version
- Installation method (micromamba + RoboStack)
- Platform info

### Detailed Output

For each test, include the captured output (first 10 lines each). Wrap outputs in code blocks.

### Errors & Notes

List any errors, warnings, or skipped tests with explanations.
