---
description: ROS2 Humble 환경을 설치해서 토픽 pub/sub 등 이것저것 테스트하고 결과를 이슈로 올립니다.

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
    title-prefix: "[ros2-vibe] "
    labels: [ros2, test-result]
---

# ROS2 Vibe Test

micromamba + RoboStack으로 ROS2 Humble 환경을 설치하고 이것저것 기능들 테스트해봐.

참고: Docker 데몬은 이 환경에서 사용 불가. GitHub 릴리즈 바이너리도 403으로 막혀있음. micromamba는 `conda.anaconda.org`에서 conda 패키지로 받아야 함.

## 하고 싶은 것

- micromamba로 ROS2 Humble(`demo_nodes_cpp`, `demo_nodes_py`) 설치
- 토픽 pub/sub 테스트 (talker/listener)
- 서비스 호출 같은 것도 해보고
- 잘 됐는지 결과를 이슈로 올려줘

테스트가 잘 됐든 안 됐든 뭔가 결과가 나오면 바로 이슈로 올려줘.
