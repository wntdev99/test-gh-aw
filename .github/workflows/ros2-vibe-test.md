---
description: ROS2 Humble을 알아서 설치하고 토픽 pub/sub 등 기능들 테스트한 뒤 결과를 이슈로 올립니다.

on:
  workflow_dispatch:

timeout-minutes: 40

permissions:
  contents: read
  issues: read

network:
  allowed:
    - defaults
    - linux-distros
    - python
    - "conda.anaconda.org"
    - "packages.ros.org"
    - "pypi.org"
    - "files.pythonhosted.org"
    - "release-assets.githubusercontent.com"
    - "python.org"
    - "www.python.org"

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

## 실패 시 해결방안 제시 지침

만약 환경 제약으로 실패했다면, 이슈에 해결방안을 제시할 때 **반드시 gh-aw `.md` 프론트매터 관점**에서 작성해줘.

예를 들어:
- `timeout-minutes` 값 증가
- `network.allowed`에 특정 도메인 또는 에코시스템 (`python`, `linux-distros` 등) 추가
- `container:` 옵션으로 ROS2가 사전 설치된 이미지 지정

일반 GitHub Actions `.yml` 설정(예: `runs-on`, `container:` job 옵션 등)을 직접 수정하라는 제안은 하지 마.
