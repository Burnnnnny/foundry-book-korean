---
description: 무작위 입력을 사용하는 속성 기반 테스트(Property-based testing)로 스마트 컨트랙트의 일반적인 동작과 엣지 케이스를 테스트합니다.
---

## 퍼즈 테스트 (Fuzz Testing)

Forge는 속성 기반 테스트(Property based testing)를 지원합니다.

속성 기반 테스트는 격리된 시나리오가 아닌 일반적인 동작을 테스트하는 방법입니다.

단위 테스트를 작성하고, 테스트하려는 일반적인 속성을 찾은 다음, 이를 속성 기반 테스트로 변환하여 그 의미를 살펴보겠습니다.

```solidity
// [!include ~/snippets/projects/fuzz_testing/test/Safe.t.sol.1:all]
```

테스트를 실행하면 통과하는 것을 볼 수 있습니다:

```bash
// [!include ~/snippets/output/fuzz_testing/forge-test-no-fuzz:all]
```

이 단위 테스트는 금고(safe)에서 이더를 인출할 수 있는지 _확인_합니다. 하지만 1 이더뿐만 아니라 모든 금액에 대해 작동한다고 누가 말할 수 있을까요?

여기서 일반적인 속성은 '금고 잔고가 주어졌을 때, 인출하면 금고에 있는 모든 것을 가져와야 한다'는 것입니다.

Forge는 하나 이상의 매개변수를 받는 모든 테스트를 속성 기반 테스트로 실행하므로 다시 작성해 보겠습니다:

```solidity
// [!include ~/snippets/projects/fuzz_testing/test/Safe.t.sol.2:contract_prelude]
    // ...

// [!include ~/snippets/projects/fuzz_testing/test/Safe.t.sol.2:test]
}
```

이제 테스트를 실행하면 Forge가 속성 기반 테스트를 실행하는 것을 볼 수 있지만, `amount` 값이 높으면 실패합니다:

```sh
forge test
// [!include ~/snippets/output/fuzz_testing/forge-test-fail-fuzz:output]
```

테스트 컨트랙트에 주어지는 이더의 기본 양은 `2**96 wei`(DappTools와 같음)이므로, 가진 것보다 더 많은 양을 보내려고 시도하지 않으려면 `amount`의 타입을 `uint96`으로 제한해야 합니다:

```solidity
// [!include ~/snippets/projects/fuzz_testing/test/Safe.t.sol.3:signature]
```

이제 통과합니다:

```sh
// [!include ~/snippets/output/fuzz_testing/forge-test-success-fuzz:all]
```

[`assume`](/reference/cheatcodes/assume) 치트코드를 사용하여 특정 케이스를 제외하고 싶을 수 있습니다. 그러한 경우, 퍼저(fuzzer)는 입력을 폐기하고 새로운 퍼즈 실행(fuzz run)을 시작합니다:

```solidity
function testFuzz_Withdraw(uint96 amount) public {
    vm.assume(amount > 0.1 ether);
    // 생략
}
```

속성 기반 테스트를 실행하는 방법에는 여러 가지가 있으며, 특히 매개변수 테스트(parametric testing)와 퍼징(fuzzing)이 있습니다. Forge는 퍼징만 지원합니다.

### 결과 해석

퍼즈 테스트는 단위 테스트와 약간 다르게 요약되는 것을 확인했을 수 있습니다:

- "runs"는 퍼저가 테스트한 시나리오의 양을 나타냅니다. 기본적으로 퍼저는 256개의 시나리오를 생성하지만, 이 값과 다른 테스트 실행 매개변수는 사용자가 설정할 수 있습니다. 퍼저 구성 세부 정보는 [`여기`](#configuring-fuzz-test-execution)에 제공됩니다.
- "μ" (그리스 문자 mu)는 모든 퍼즈 실행에서 사용된 평균 가스입니다.
- "~" (틸드)는 모든 퍼즈 실행에서 사용된 가스의 중간값(median)입니다.

### 퍼즈 테스트 실행 구성

퍼즈 테스트 실행은 사용자가 Forge 구성 프리미티브를 통해 제어할 수 있는 매개변수에 의해 관리됩니다. 구성은 전역적으로 또는 테스트별로 적용할 수 있습니다. 이 주제에 대한 자세한 내용은 📚 [`전역 구성`](/config/reference/overview) 및 📚 [`인라인 구성`](/config/reference/inline-test-config)을 참조하세요.

#### 퍼즈 테스트 픽스처 (Fixtures)

퍼즈 테스트 픽스처는 특정 값 세트가 퍼즈 매개변수의 입력으로 사용되도록 하려는 경우에 정의할 수 있습니다.
이러한 픽스처는 테스트에서 다음과 같이 선언할 수 있습니다:

- `fixture` 접두사와 퍼징될 매개변수 이름이 뒤따르는 스토리지 배열. 예를 들어, `uint32` 타입의 `amount` 매개변수를 퍼징할 때 사용할 픽스처는 다음과 같이 정의할 수 있습니다.

```solidity
uint32[] public fixtureAmount = [1, 5, 555];
```

- `fixture` 접두사와 퍼징될 매개변수 이름이 뒤따르는 함수. 함수는 퍼징에 사용될 값의 배열(고정 크기 또는 동적)을 반환해야 합니다. 예를 들어, `address` 타입의 `owner`라는 매개변수를 퍼징할 때 사용할 픽스처는 다음과 같은 서명을 가진 함수로 정의할 수 있습니다.

```solidity
function fixtureOwner() public returns (address[] memory)
```

픽스처로 제공된 값의 타입이 퍼징될 명명된 매개변수의 타입과 같지 않으면 거부되고 오류가 발생합니다.

픽스처가 사용될 수 있는 예로는 `DSChief` 취약점을 재현하는 경우가 있습니다. 다음 2개의 함수를 고려해 보세요:

```solidity
    function etch(address yay) public returns (bytes32 slate) {
        bytes32 hash = keccak256(abi.encodePacked(yay));

        slates[hash] = yay;

        return hash;
    }

    function voteSlate(bytes32 slate) public {
        uint weight = deposits[msg.sender];
        subWeight(weight, votes[msg.sender]);
        votes[msg.sender] = slate;
        addWeight(weight, votes[msg.sender]);
    }
```

여기서 취약점은 `etch` 전에 `voteSlate`를 호출하여 재현할 수 있으며, `slate` 값은 `yay` 주소의 해시입니다.
퍼저가 동일한 실행에 `yay` 주소에서 파생된 `slate` 값을 포함하도록 하려면 다음 픽스처를 정의할 수 있습니다:

```solidity
    address[] public fixtureYay = [
        makeAddr("yay1"),
        makeAddr("yay2"),
        makeAddr("yay3")
    ];

    bytes32[] public fixtureSlate = [
        keccak256(abi.encodePacked(makeAddr("yay1"))),
        keccak256(abi.encodePacked(makeAddr("yay2"))),
        keccak256(abi.encodePacked(makeAddr("yay3")))
    ];
```

`fixtureYay`라는 이름은 `etch`의 `yay` 매개변수와 일치하고 `fixtureSlate`는 `voteSlate`의 `slate` 매개변수와 일치한다는 점에 유의하세요.

다음 이미지는 픽스처가 선언되었을 때와 선언되지 않았을 때 퍼저가 값을 생성하는 방법을 보여줍니다:

![퍼저](/fuzzer.png)
