---
description: 치트코드를 사용하여 블록체인 상태를 조작하고, 리버트(revert)를 테스트하고, 스마트 컨트랙트 테스트에서 이벤트를 검증합니다.
---

## 치트코드 (Cheatcodes)

대부분의 경우, 단순히 스마트 컨트랙트 출력을 테스트하는 것만으로는 충분하지 않습니다. 블록체인의 상태를 조작하고 특정 리버트(revert) 및 이벤트를 테스트하기 위해 Foundry는 일련의 치트코드(cheatcodes)를 제공합니다.

치트코드를 사용하면 블록 번호, 신원 등을 변경할 수 있습니다. 치트코드는 특별히 지정된 주소인 `0x7109709ECfa91a80626fF3989D68f67F5b1DD12D`의 특정 함수를 호출하여 실행됩니다.

Forge 표준 라이브러리의 `Test` 컨트랙트에서 사용할 수 있는 `vm` 인스턴스를 통해 치트코드에 쉽게 접근할 수 있습니다. Forge 표준 라이브러리는 다음 [섹션](/forge/tests/forge-std)에서 자세히 설명합니다.

소유자(owner)만 호출할 수 있는 스마트 컨트랙트에 대한 테스트를 작성해 보겠습니다.

```solidity
// [!include ~/snippets/projects/cheatcodes/test/OwnerUpOnly.t.sol:prelude]

// [!include ~/snippets/projects/cheatcodes/test/OwnerUpOnly.t.sol:contract]

// [!include ~/snippets/projects/cheatcodes/test/OwnerUpOnly.t.sol:contract_prelude]

// [!include ~/snippets/projects/cheatcodes/test/OwnerUpOnly.t.sol:simple_test]
}
```

이제 `forge test`를 실행하면, `OwnerUpOnlyTest`가 `OwnerUpOnly`의 소유자이므로 테스트가 통과하는 것을 볼 수 있습니다.

```sh
forge test
// [!include ~/snippets/output/cheatcodes/forge-test-simple:output}]
```

이제 `expectRevert` 치트코드를 사용하여 소유자가 아닌 사람이 카운트를 증가시킬 수 없도록 해보겠습니다:

```solidity
// [!include ~/snippets/projects/cheatcodes/test/OwnerUpOnly.t.sol:contract_prelude]

    // ...

// [!include ~/snippets/projects/cheatcodes/test/OwnerUpOnly.t.sol:test_expectrevert]
}
```

마지막으로 `forge test`를 실행하면 테스트가 여전히 통과하는 것을 볼 수 있지만, 이번에는 다른 이유로 리버트되면 항상 실패할 것임을 확신할 수 있습니다.

```bash
forge test
// [!include ~/snippets/output/cheatcodes/forge-test-cheatcodes-expectrevert:output]
```

직관적이지 않을 수 있는 또 다른 치트코드는 `expectEmit` 함수입니다. `expectEmit`을 살펴보기 전에 이벤트가 무엇인지 이해해야 합니다.

이벤트는 컨트랙트의 상속 가능한 멤버입니다. 이벤트를 발생시키면 인수가 블록체인에 저장됩니다. `indexed` 속성은 최대 3개의 파라미터에 추가하여 "토픽(topic)"이라는 데이터 구조를 형성할 수 있습니다. 토픽을 통해 사용자는 블록체인에서 이벤트를 검색할 수 있습니다.

```solidity
// [!include ~/snippets/projects/cheatcodes/test/EmitContract.t.sol:all]
```

`vm.expectEmit(true, true, false, true);`를 호출할 때, 우리는 다음 이벤트에 대해 1번째와 2번째 `indexed` 토픽을 확인하고자 합니다.

`test_ExpectEmit()`의 예상되는 `Transfer` 이벤트는 `from`이 `address(this)`이고, `to`가 `address(1337)`일 것을 기대한다는 의미입니다. 이는 `emitter.t()`에서 실제로 발생한 이벤트와 비교됩니다.

즉, 우리는 `emitter.t()`의 첫 번째 토픽이 `address(this)`와 같은지 확인하는 것입니다. `expectEmit`의 3번째 인수는 `false`로 설정되어 있는데, `Transfer` 이벤트에는 토픽이 두 개뿐이므로 세 번째 토픽을 확인할 필요가 없기 때문입니다. `true`로 설정해도 상관없습니다.

`expectEmit`의 4번째 인수는 `true`로 설정되어 있는데, 이는 "인덱스되지 않은 토픽", 즉 데이터(data)를 확인하고자 함을 의미합니다.

예를 들어, 우리는 `test_ExpectEmit`의 예상 이벤트 데이터(즉, `amount`)가 실제 발생한 이벤트의 데이터와 같기를 원합니다. 다시 말해, 우리는 `emitter.t()`에 의해 발생한 `amount`가 `1337`과 같다고 단언(assert)하는 것입니다. 만약 `expectEmit`의 4번째 인수가 `false`로 설정되었다면 `amount`를 확인하지 않았을 것입니다.

즉, 데이터를 확인하지 않으므로 금액이 다르더라도 `test_ExpectEmit_DoNotCheckData`는 유효한 테스트 케이스입니다.

<br></br>

:::info
사용 가능한 모든 치트코드에 대한 전체 개요는 [치트코드 참조](/reference/cheatcodes/overview)를 확인하세요.
:::
