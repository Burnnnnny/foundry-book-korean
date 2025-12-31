---
description: 단위 테스트, 포크 테스트, 테스트 구성 및 스마트 컨트랙트를 위한 하네스 패턴을 포함한 포괄적인 테스트 전략입니다.
---

## 테스트

---

### 일반적인 테스트 지침

#### 테스트 파일 명명

`MyContract.sol`을 테스트하는 경우 테스트 파일은 `MyContract.t.sol`이어야 합니다. `MyScript.s.sol`을 테스트하는 경우 테스트 파일은 `MyScript.t.sol`이어야 합니다.

- 컨트랙트가 크고 여러 파일로 나누고 싶다면 `MyContract.owner.t.sol`, `MyContract.deposits.t.sol` 등과 같이 논리적으로 그룹화하세요.

#### `setUp`에 어설션(assertions) 없음

`setUp` 함수에서 절대 어설션을 수행하지 마세요. `setUp` 함수가 예상대로 수행되는지 확인해야 하는 경우 `test_SetUpState()`와 같은 전용 테스트를 사용하세요. 그 이유에 대한 자세한 내용은 [foundry-rs/foundry#1291](https://github.com/foundry-rs/foundry/issues/1291)을 참조하세요.

#### 테스트 구성 및 명명

단위 테스트의 경우 테스트를 구성하는 두 가지 주요 방법이 있습니다:

1.  컨트랙트를 describe 블록으로 취급:

    - `contract Add`는 `MyContract`의 `add` 메서드에 대한 모든 단위 테스트를 보유합니다.
    - `contract Supply`는 `supply` 메서드에 대한 모든 테스트를 보유합니다.
    - `contract Constructor`는 생성자에 대한 모든 테스트를 보유합니다.
    - 이 접근 방식의 장점은 작은 컨트랙트가 큰 컨트랙트보다 빠르게 컴파일되어야 하므로, 테스트 스위트가 커짐에 따라 많은 작은 컨트랙트로 구성하는 방식이 시간을 절약할 수 있다는 것입니다.

2.  테스트 대상 컨트랙트당 하나의 테스트 컨트랙트를 갖고, 원하는 만큼 유틸리티와 픽스처를 보유:

    - `contract MyContractTest`는 `MyContract`에 대한 모든 단위 테스트를 보유합니다.
    - `function test_add_AddsTwoNumbers()`는 `add` 메서드를 테스트하기 위해 `MyContractTest` 내에 존재합니다.
    - `function test_supply_UsersCanSupplyTokens()`도 `supply` 메서드를 테스트하기 위해 `MyContractTest` 내에 존재합니다.
    - 이 접근 방식의 장점은 테스트 출력이 테스트 대상 컨트랙트별로 그룹화되어 실패 위치를 빠르게 파악하기 쉽다는 것입니다.

3.  모든 종류의 테스트에 대한 몇 가지 일반적인 지침:

    - 테스트 컨트랙트/함수는 테스트 대상 컨트랙트의 원래 함수와 동일한 순서로 작성해야 합니다.
    - 동일한 함수를 테스트하는 모든 단위 테스트는 테스트 파일 내에서 연속적으로 위치해야 합니다.
    - 테스트 컨트랙트는 원하는 도우미 컨트랙트에서 상속받을 수 있습니다. 예를 들어 `contract MyContractTest`는 `MyContract`를 테스트하지만, forge-std의 `Test`나 자신의 `TestUtilities` 도우미 컨트랙트에서 상속받을 수 있습니다.

4.  통합 테스트는 동일한 `test` 디렉토리에 명확한 명명 규칙을 가지고 있어야 합니다. 전용 파일에 있거나 기존 테스트 파일의 관련 단위 테스트 옆에 있을 수 있습니다.

5.  테스트 명명은 일관성을 유지하세요. 이는 테스트 필터링에 도움이 됩니다(예: 가스 보고서의 경우 되돌리기(revert) 테스트를 필터링하고 싶을 수 있음). 명명 규칙을 결합할 때는 알파벳 순서로 유지하세요. 아래는 유효한 이름의 예시입니다. 유효하거나 유효하지 않은 예시의 전체 목록은 [여기](https://github.com/ScopeLift/scopelint/blob/1857e3940bfe92ac5a136827374f4b27ff083971/src/check/validators/test_names.rs#L106-L143)에서 확인할 수 있습니다.

    - 표준 테스트의 경우 `test_Description`.
    - 퍼즈 테스트의 경우 `testFuzz_Description`.
    - 되돌리기가 예상되는 테스트의 경우 `test_Revert[If|When]_Condition`.
    - 네트워크에서 포크하는 테스트의 경우 `testFork_Description`.
    - 포크하고 되돌리기를 예상하는 퍼즈 테스트의 경우 `testForkFuzz_Revert[If|When]_Condition`.

6.  상수와 불변 변수의 이름은 `ALL_CAPS_WITH_UNDERSCORES`를 사용하여 변수 및 함수와 구별하기 쉽게 만드세요.

7.  `assertEq`와 같은 어설션을 사용할 때 마지막 문자열 매개변수를 활용하여 실패를 식별하기 쉽게 만드는 것을 고려하세요. 이들은 간결하게 유지하거나 단순히 숫자일 수도 있습니다. 기본적으로 되돌리기의 줄 번호를 표시하는 것을 대체하는 역할을 합니다. 예: `assertEq(x, y, "1")` 또는 `assertEq(x, y, "sum1")`. _(참고: [foundry-rs/foundry#2328](https://github.com/foundry-rs/foundry/issues/2328)에서 이를 기본적으로 통합하는 것을 추적 중입니다)._

8.  이벤트를 테스트할 때 모든 `expectEmit` 인수를 `true`로 설정하는 것을 선호하세요. 즉, `vm.expectEmit(true, true, true, true)` 또는 `vm.expectEmit()`. 장점:

    - 이벤트의 모든 것을 테스트하도록 보장합니다.
    - 토픽(즉, 새로운 인덱싱된 매개변수)을 추가하면 기본적으로 테스트됩니다.
    - 토픽이 1개만 있더라도 추가적인 `true` 인수는 해가 되지 않습니다.

9.  불변성 테스트(invariant tests)를 작성하는 것을 잊지 마세요! 어설션 문자열에는 불변성에 대한 자세한 영어 설명을 사용하세요: `assertEq(x + y, z, "Invariant violated: the sum of x and y must always equal z")`.

### 포크 테스트 (Fork Tests)

#### 포크 테스트를 자유롭게 사용하세요

포크 테스트를 특별하게 취급할 필요는 없으며 자유롭게 사용하세요:

- 비공개 소스 web2 개발에서는 모의(Mock)가 _필수_ 입니다. API 코드가 오픈 소스가 아니어서 로컬에서 실행할 수 없으므로 API 응답을 모의해야 합니다. 하지만 블록체인에서는 그렇지 않습니다. 이미 배포된 상호 작용하는 모든 코드는 로컬에서 무료로 실행하고 수정할 수도 있습니다. 굳이 필요하지 않은데 잘못된 모의의 위험을 도입할 이유가 있을까요?
- 포크 테스트를 피하고 모의를 선호하는 일반적인 이유는 포크 테스트가 느리다는 것입니다. 하지만 이것이 항상 사실은 아닙니다. 블록 번호를 고정(pinning)하면 forge가 RPC 응답을 캐시하므로 첫 번째 실행만 느리고 후속 실행은 훨씬 빠릅니다. [이 벤치마크](https://github.com/mds1/convex-shutdown-simulation/)를 보면, 원격 RPC를 사용한 첫 번째 실행에는 7분이 걸렸지만 데이터가 캐시된 후에는 0.5초밖에 걸리지 않았습니다. [Alchemy](https://alchemy.com), [Infura](https://infura.io) 및 [Tenderly](https://tenderly.co)는 무료 아카이브 데이터를 제공하므로 블록에 고정하는 것은 문제가 되지 않습니다.
- [foundry-toolchain](https://github.com/foundry-rs/foundry-toolchain) GitHub Action은 CI에서 기본적으로 RPC 응답을 캐시하며, 포크 테스트를 업데이트할 때 캐시도 업데이트합니다.

#### RPC 요청 최소화

비결정적 퍼징으로 RPC 요청을 낭비하지 않도록 포크에서의 퍼즈 테스트에 주의하세요. 포크 퍼즈 테스트의 입력이 데이터를 가져오기 위해 RPC 호출에 사용되는 매개변수(예: 주소의 토큰 잔액 조회)인 경우, 퍼즈 테스트를 실행할 때마다 최소 1개의 RPC 요청을 사용하므로 속도 제한이나 사용량 제한에 빠르게 도달할 수 있습니다. 고려할 솔루션:

- 여러 RPC 호출을 단일 [multicall](https://github.com/mds1/multicall)로 대체하세요.
- 퍼즈/불변성 [시드](/config/reference/testing#seed)를 지정하세요: 이는 각 `forge test` 호출이 동일한 퍼즈 입력을 사용하도록 보장합니다. RPC 결과는 로컬에 캐시되므로 처음 한 번만 노드를 쿼리하게 됩니다.
- CI에서는 [계산된 환경 변수](https://github.com/sablier-labs/v2-core/blob/d1157b49ed4bceeff0c4e437c9f723e88c134d3a/.github/workflows/ci.yml#L252-L254)를 사용하여 퍼즈 시드를 설정하여 매일 또는 매주 변경되도록 하는 것을 고려하세요. 이는 더 많은 버그를 찾기 위한 무작위성 증가와 RPC 요청 감소를 위한 시드 사용 사이의 균형을 맞출 수 있는 유연성을 제공합니다.
- 퍼징하는 데이터가 RPC 호출에 사용되는 데이터가 아니라 컨트랙트에 의해 로컬에서 계산되도록 테스트를 구조화하세요(수행하는 작업에 따라 가능할 수도 있고 불가능할 수도 있음).
- 마지막으로, 물론 항상 로컬 노드를 실행하거나 RPC 요금제를 업그레이드할 수 있습니다.

#### `foundry.toml`에서 포크 url 구성 및 치트코드 사용

1. 포크 테스트를 작성할 때 `--fork-url` 플래그를 사용하지 마세요. 대신 향상된 유연성을 위해 다음 접근 방식을 선호하세요:

   - `foundry.toml` 구성 파일에 `[rpc_endpoints]`를 정의하고 [포킹 치트코드](/forge/tests/fork-testing#forking-cheatcodes)를 사용하세요.
   - forge-std의 `stdChains.ChainName.rpcUrl`을 사용하여 테스트에서 RPC URL 엔드포인트에 액세스하세요. 지원되는 체인 목록과 예상되는 구성 파일 별칭은 [여기](https://github.com/foundry-rs/forge-std/blob/ff4bf7db008d096ea5a657f2c20516182252a3ed/src/StdCheats.sol#L255-L271)를 참조하세요.
   - 테스트가 결정적이고 RPC 응답이 캐시되도록 항상 블록에 고정하세요.
   - 이 포크 테스트 접근 방식에 대한 자세한 정보는 [여기](https://twitter.com/msolomon44/status/1564742781129502722)에서 확인할 수 있습니다 (`StdChains` 이전 내용이므로 해당 측면은 약간 구식입니다).

### 테스트 하네스 (Test Harnesses)

#### 내부 함수 (Internal Functions)

`internal` 함수를 테스트하려면 테스트 대상 컨트랙트(CuT)를 상속받는 하네스 컨트랙트를 작성하세요. CuT를 상속받는 하네스 컨트랙트는 `internal` 함수를 `external` 함수로 노출합니다.

테스트되는 각 `internal` 함수는 `exposed<FunctionName>` 패턴을 따르는 이름의 외부 함수를 통해 노출되어야 합니다. 예를 들어:

```solidity
// file: src/MyContract.sol
contract MyContract {
  function myInternalMethod() internal returns (uint) {
    return 42;
  }
}

// file: test/MyContract.t.sol
import {MyContract} from "src/MyContract.sol";

contract MyContractHarness is MyContract {
  // 이 컨트랙트를 배포한 다음 이 메서드를 호출하여 `myInternalMethod`를 테스트합니다.
  function exposedMyInternalMethod() external returns (uint) {
    return myInternalMethod();
  }
}
```

#### 비공개 함수 (Private Functions)

불행히도 `private` 메서드는 다른 컨트랙트에서 액세스할 수 없으므로 현재 단위 테스트할 수 있는 좋은 방법이 없습니다. 옵션은 다음과 같습니다:

- `private` 함수를 `internal`로 변환합니다.
- 로직을 테스트 컨트랙트에 복사/붙여넣기하고 CI 체크에서 실행되는 스크립트를 작성하여 두 함수가 동일한지 확인합니다.

#### 해결책 함수 (Workaround Functions)

하네스는 원래 스마트 컨트랙트에서 사용할 수 없는 기능이나 정보를 노출하는 데에도 사용할 수 있습니다. 가장 간단한 예는 공개 배열의 길이를 테스트하려는 경우입니다. 함수는 `workaround_<function_name>` 패턴을 따라야 합니다(예: `workaround_queueLength()`).

이에 대한 또 다른 사용 사례는 불변성 테스트를 돕기 위해 프로덕션에서는 추적하지 않을 데이터를 추적하는 것입니다. 예를 들어 "모든 잔액의 합계는 총 공급량과 같아야 한다"는 불변성 검증을 단순화하기 위해 모든 토큰 보유자 목록을 저장할 수 있습니다. 이는 종종 "고스트 변수(ghost variables)"로 알려져 있습니다. [Rikard Hjort](https://twitter.com/rikardhjort)의 [실무 DeFi 개발자를 위한 형식적 방법론](https://youtu.be/ETlNhV9jYJw) 강연에서 이에 대해 자세히 알아볼 수 있습니다.
