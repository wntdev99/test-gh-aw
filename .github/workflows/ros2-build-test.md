---
description: ROS2 talker/listener 테스트 후 결과를 GitHub Issue로 보고합니다.

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

# ROS2 Test

Run the following bash script exactly as written, then **immediately call `create_issue`** with the results regardless of success or failure.

## Script

```bash
set -e
export MAMBA_ROOT_PREFIX="$HOME/micromamba"
mkdir -p "$MAMBA_ROOT_PREFIX" /tmp/mamba_extract

# 1. Get micromamba from conda-forge (github.com/releases is blocked - use conda.anaconda.org)
PKG=$(python3 - <<'EOF'
import urllib.request, json, bz2
url = "https://conda.anaconda.org/conda-forge/linux-64/repodata.json.bz2"
data = json.loads(bz2.decompress(urllib.request.urlopen(url).read()))
pkgs = sorted([(k,v) for k,v in data['packages'].items() if v['name']=='micromamba'],
              key=lambda x: [int(i) for i in x[1]['version'].split('.')], reverse=True)
print(pkgs[0][0])
EOF
)
curl -fsSL "https://conda.anaconda.org/conda-forge/linux-64/$PKG" | tar -xj -C /tmp/mamba_extract bin/micromamba
MM=/tmp/mamba_extract/bin/micromamba
chmod +x "$MM"

# 2. Install ROS2 Humble demo nodes
$MM create -n ros_env -c conda-forge -c robostack-staging --root-prefix "$MAMBA_ROOT_PREFIX" --no-rc -y \
  ros-humble-demo-nodes-cpp ros-humble-demo-nodes-py

# 3. Activate
eval "$($MM shell activate -s bash -n ros_env --root-prefix "$MAMBA_ROOT_PREFIX")"

# 4. talker/listener test
ros2 run demo_nodes_cpp talker > /tmp/talker.txt 2>&1 &
TPID=$!
sleep 3
timeout 5 ros2 run demo_nodes_cpp listener > /tmp/listener.txt 2>&1 || true
kill $TPID 2>/dev/null; wait $TPID 2>/dev/null

TALKER=$(cat /tmp/talker.txt | head -10)
LISTENER=$(cat /tmp/listener.txt | head -10)
echo "$LISTENER" | grep -q "Hello World" && RESULT=PASS || RESULT=FAIL
```

## Issue Creation

After running the script (success or failure), call `create_issue` with:

- Title: `Talker/Listener - PASS` or `Talker/Listener - FAIL`
- Body: result summary, talker output, listener output, and any errors encountered
