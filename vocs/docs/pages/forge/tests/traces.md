---
description: 가스 사용량, 컨트랙트 상호작용 및 반환 값을 보여주는 상세한 호출 트레이스로 테스트 실행을 디버그합니다.
---

## 트레이스(Traces) 이해하기

Forge는 실패한 테스트(`-vvv`) 또는 모든 테스트(`-vvvv`)에 대해 트레이스를 생성할 수 있습니다.

트레이스는 다음과 같은 일반적인 형식을 따릅니다:

```
  [<Gas Usage>] <Contract>::<Function>(<Parameters>)
    ├─ [<Gas Usage>] <Contract>::<Function>(<Parameters>)
    │   └─ ← <Return Value>
    └─ ← <Return Value>
```

각 트레이스에는 더 많은 하위 트레이스가 있을 수 있으며, 각 하위 트레이스는 컨트랙트 호출과 반환 값을 나타냅니다.

터미널이 색상을 지원하는 경우, 트레이스는 다양한 색상으로 표시됩니다:

- **녹색**: 리버트되지 않은 호출
- **빨간색**: 리버트되는 호출
- **파란색**: 치트코드 호출
- **청록색**: 발생된(emitted) 로그
- **노란색**: 컨트랙트 배포

가스 사용량(대괄호로 표시됨)은 전체 함수 호출에 대한 것입니다. 하지만 가끔 하나의 트레이스의 가스 사용량이 모든 하위 트레이스의 가스 사용량과 정확히 일치하지 않는 것을 볼 수 있습니다:

```
  [24661] OwnerUpOnlyTest::testIncrementAsOwner()
    ├─ [2262] OwnerUpOnly::count()
    │   └─ ← 0
    ├─ [20398] OwnerUpOnly::increment()
    │   └─ ← ()
    ├─ [262] OwnerUpOnly::count()
    │   └─ ← 1
    └─ ← ()
```

설명되지 않는 가스는 산술 연산 및 스토리지 읽기/쓰기와 같이 호출 사이에 발생하는 일부 추가 작업 때문입니다.

Forge는 가능한 한 많은 서명과 값을 디코딩하려고 시도하지만 때로는 불가능할 수 있습니다. 이러한 경우 트레이스는 다음과 같이 표시됩니다:

```
  [<Gas Usage>] <Address>::<Calldata>
    └─ ← <Return Data>
```

어떤 트레이스는 언뜻 보기에 이해하기 어려울 수 있습니다. 다음은 그 예입니다:

- `OOG`는 "Out Of Gas"(가스 부족)의 약어입니다.
- `EOF`는 "Ethereum Object Format"의 약어로, EVM 바이트코드를 위한 확장 가능하고 버전이 지정된 컨테이너 형식을 도입합니다. 자세한 내용은 [여기](https://evmobjectformat.org/)를 참조하세요.
- `NotActivated`는 기능이나 오퍼코드가 활성화되지 않았음을 의미합니다. EVM의 일부 버전은 특정 오퍼코드만 지원합니다. `--evm_version` 플래그를 사용하여 더 최신 버전을 사용해야 할 수도 있습니다. 예를 들어, `PUSH0` 오퍼코드는 [Shanghai](https://www.evm.codes/?fork=shanghai) 하드포크 이후부터만 사용할 수 있습니다.
- `InvalidFEOpcode`는 실행 중에 정의되지 않은 바이트코드 값을 만났음을 의미합니다. EVM은 알 수 없는 바이트코드를 포착하고 대신 `0xFE` 값의 `INVALID` 오퍼코드를 반환합니다. 자세한 내용은 [여기](https://www.evm.codes/#fe)에서 확인할 수 있습니다.

다양한 트레이스에 대한 더 깊은 통찰력을 얻으려면 [revm 소스 코드](https://github.com/bluealloy/revm/blob/main/crates/interpreter/src/instruction_result.rs)를 탐색해 보세요.
