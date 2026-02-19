---
description: ROS2 Humble을 알아서 설치하고 토픽 pub/sub 등 기능들 테스트한 뒤 결과를 이슈로 올립니다.

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

현재 환경을 먼저 파악하고, ROS2 Humble을 설치할 수 있는 적절한 방법을 스스로 찾아서 설치해봐.

## 원하는 것

ROS2 Humble이 돌아가는 환경을 만들어서 토픽 pub/sub이나 서비스 호출 같은 기능들을 좀 테스트해보고, 결과를 이슈로 올려줘.

잘 됐든 안 됐든 결과가 나오면 바로 이슈로 올려줘.
