---
description: 퍼즈 테스트, 불변성 테스트 및 단위 테스트에 대해 지속된 실패 데이터를 사용하여 점진적으로 테스트 실패를 다시 재생합니다.
---

## 실패 재생 (Replaying Failures)

Forge는 디스크에 지속(persist)시키는 방식으로 지난 테스트 실행 실패를 점진적으로 다시 재생하는 것을 지원합니다.

### 실패 다시 실행

`--rerun` 옵션을 사용하여 성공한 테스트를 생략하고 기록된 실패만 다시 재생할 수 있습니다:

```bash
forge test --rerun
```

실패한 테스트는 `~/.foundry/cache/test-failures` 파일에 기록됩니다. 이 파일은 `forge test`가 수행될 때마다 업데이트되므로 마지막 실행의 실패를 반영합니다.

### 퍼즈 테스트 실패

Forge는 모든 퍼즈 테스트 반례(counterexamples)를 저장하고 새로운 테스트 캠페인이 시작되기 전에 다시 재생합니다 (이는 회귀가 발생하지 않도록 하기 위함입니다).
여러 실행에서 발생한 퍼즈 테스트 실패는 기본적으로 `~/.foundry/cache/fuzz/failures` 파일에 지속됩니다. 이 파일의 내용은 후속 테스트 실행에 의해 대체되지 않으며, 기존 항목에 새 레코드가 추가됩니다.

퍼즈 테스트 실패를 지속하고 다시 실행하는 데 사용되는 기본 파일은 `foundry.toml`에서 변경할 수 있습니다:

```toml
[fuzz]
failure_persist_file="/tests/failures.txt"
```

또는 인라인 구성을 사용하여 변경할 수 있습니다:

```solidity
/// forge-config: default.fuzz.failure-persist-file = /tests/failures.txt
```

### 불변성 테스트 실패

불변성 테스트 실패는 퍼즈 테스트와 유사하게 저장되고 새로운 테스트 캠페인이 시작되기 전에 다시 재생됩니다. 차이점은 실패한 시퀀스가 개별 파일에 지속되며, 기본 경로는 `~/.foundry/cache/invariant/failures/{TEST_SUITE_NAME}/{INVARIANT_NAME}`라는 것입니다. 이 파일의 내용은 다른 반례가 발견될 때만 대체됩니다.

불변성 테스트 실패를 지속하는 기본 디렉토리는 `foundry.toml`에서 변경할 수 있습니다:

```toml
[invariant]
failure_persist_dir="/tests/dir"
```

또는 인라인 구성을 사용하여 변경할 수 있습니다:

```Solidity
/// forge-config: default.invariant.failure-persist-dir = /tests/dir
```

### 지속된 실패 제거

저장된 실패를 무시하고 깨끗한 테스트 캠페인을 시작하려면, 단순히 지속된 파일을 제거하거나 [`forge clean`](/forge/reference/clean)(모든 빌드 아티팩트 및 캐시 디렉토리 제거)을 실행하면 됩니다.
