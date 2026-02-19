---
description: Docker로 ROS2 Humble 환경을 띄워서 토픽 pub/sub 등 이것저것 테스트하고 결과를 이슈로 올립니다.

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

Docker로 ROS2 Humble 컨테이너를 띄워서 이것저것 기능들 테스트해봐.

## 하고 싶은 것

- `osrf/ros:humble-ros-base` 이미지로 컨테이너 실행
- 그 안에서 `demo_nodes_cpp` 패키지로 토픽 pub/sub 테스트
- 서비스 호출 같은 것도 해보고
- 잘 됐는지 결과를 이슈로 올려줘

테스트가 잘 됐든 안 됐든 뭔가 결과가 나오면 바로 이슈로 올려줘.
