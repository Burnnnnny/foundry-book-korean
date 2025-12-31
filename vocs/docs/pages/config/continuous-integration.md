## 지속적 통합 (CI)

### GitHub Actions

GitHub Actions를 사용하여 프로젝트를 테스트하려면, 다음은 워크플로우 예시입니다:

```yml
on: [push]

name: test

jobs:
  check:
    name: Foundry project
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Install Foundry
        uses: foundry-rs/foundry-toolchain@v1
        with:
          version: stable

      - name: Run tests
        run: forge test -vvv
```

### Travis CI

Travis CI를 사용하여 프로젝트를 테스트하려면, 다음은 워크플로우 예시입니다:

```yml
language: rust
cache:
  cargo: true
  directories:
    - $HOME/.foundry

install:
  - curl -L https://foundry.paradigm.xyz | bash
  - export PATH=$PATH:$HOME/.foundry/bin
  - foundryup -b master

script:
  - forge test -vvv
```

## GitLab CI

GitLab CI를 사용하여 프로젝트를 테스트하려면, 다음은 워크플로우 예시입니다:
참고: 원격 이미지를 가져오려면 [정책(Policy)](https://docs.gitlab.com/runner/executors/docker.html#how-pull-policies-work)을 확인하세요.

```yml
variables:
  GIT_SUBMODULE_STRATEGY: recursive

jobs:
  image: ghcr.io/foundry-rs/foundry
  script:
    - forge install
    - forge test -vvv
```
