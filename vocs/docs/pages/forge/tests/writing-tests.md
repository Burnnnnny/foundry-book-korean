---
description: 테스트 함수, 설정 패턴 및 공유 컨트랙트가 있는 Forge 표준 라이브러리를 사용하여 Solidity 테스트를 작성하는 방법을 배웁니다.
---

## 테스트 작성

테스트는 Solidity로 작성됩니다. 테스트 함수가 되돌려지면(revert) 테스트가 실패하고, 그렇지 않으면 통과합니다.

Forge로 테스트를 작성하는 가장 권장되는 방법인 [Forge 표준 라이브러리(Forge Standard Library)](https://github.com/foundry-rs/forge-std)의 `Test` 컨트랙트를 사용하여 테스트를 작성하는 가장 일반적인 방법을 살펴보겠습니다.

이 섹션에서는 [DSTest](https://github.com/dapphub/ds-test)의 상위 집합(superset)인 Forge Std의 `Test` 컨트랙트에 있는 함수를 사용하여 기본 사항을 다룹니다. Forge 표준 라이브러리의 더 고급 기능은 [곧](/forge/tests/forge-std) 배우게 될 것입니다.

DSTest는 기본적인 로깅 및 어설션(assertion) 기능을 제공합니다. 함수에 접근하려면 `forge-std/Test.sol`을 가져오고 테스트 컨트랙트에서 `Test`를 상속하세요:

```solidity
// [!include ~/snippets/projects/writing_tests/test/Basic.t.sol:import]
```

기본적인 테스트를 살펴보겠습니다:

```solidity
// [!include ~/snippets/projects/writing_tests/test/Basic.t.sol:all]
```

Forge는 테스트에서 다음 키워드를 사용합니다:

- `setUp`: 각 테스트 케이스가 실행되기 전에 호출되는 선택적 함수입니다.

```solidity
// [!include ~/snippets/projects/writing_tests/test/Basic.t.sol:setUp]
```

- `test`: `test`로 시작하는 함수는 테스트 케이스로 실행됩니다.

```solidity
// [!include ~/snippets/projects/writing_tests/test/Basic.t.sol:testNumberIs42]
```

좋은 관행은 [`expectRevert`](/reference/cheatcodes/expect-revert) 치트코드(치트코드는 다음 [섹션](/forge/tests/cheatcodes)에서 자세히 설명)와 함께 `test_Revert[If|When]_Condition` 패턴을 사용하는 것입니다. 또한 다른 테스트 관행은 [가이드 섹션](/guides/best-practices/writing-tests)에서 찾을 수 있습니다.

> **참고**: `stdError` 상수(아래 예제의 `arithmeticError` 같은)를 사용하려면 `StdError.sol`을 가져와야 합니다:
>
> ```solidity
> import {stdError} from "forge-std/StdError.sol";
> ```

이런 방식으로 무엇이 되돌려졌고 어떤 오류가 발생했는지 정확히 알 수 있습니다:

```solidity
// [!include ~/snippets/projects/writing_tests/test/Basic2.t.sol:testCannotSubtract43]
```

<br></br>

테스트는 `0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84`에 배포됩니다. 테스트 내에서 컨트랙트를 배포하면 `0xb4c...7e84`가 배포자가 됩니다. 테스트 내에서 배포된 컨트랙트가 `Ownable.sol`의 `onlyOwner` 수정자와 같이 배포자에게 특별한 권한을 부여하는 경우, 테스트 컨트랙트 `0xb4c...7e84`는 해당 권한을 갖게 됩니다.

> ⚠️ **참고**
>
> 테스트 함수는 `external` 또는 `public` 가시성을 가져야 합니다. `internal` 또는 `private`으로 선언된 함수는 `test`로 시작하더라도 Forge에서 인식되지 않습니다.

### 테스트 설정 전 단계 (Before test setups)

단위 및 퍼즈 테스트는 상태가 없으며(stateless) 단일 트랜잭션으로 실행됩니다. 즉, 한 테스트에서 수정된 상태는 다른 테스트에서 사용할 수 없습니다(대신 `setUp` 호출에 의해 생성된 동일한 상태를 사용합니다).
`beforeTestSetup` 함수를 구현하여 의존성 트리가 있는 단일 테스트에서 여러 트랜잭션을 시뮬레이션할 수 있습니다.

- `beforeTestSetup`: 테스트 전에 실행할 일련의 트랜잭션을 구성하는 선택적 함수입니다.

```solidity
function beforeTestSetup(
    bytes4 testSelector
) public returns (bytes[] memory beforeTestCalldata)
```

여기서

- `bytes4 testSelector`는 트랜잭션이 적용되는 테스트의 선택자입니다.
- `bytes[] memory beforeTestCalldata`는 테스트 실행 전에 적용되는 임의의 호출 데이터 배열입니다.

> 💡 **팁**
>
> 이 설정은 테스트를 체이닝하거나 테스트 실행 전에 특정 트랜잭션이 커밋되어야 하는 시나리오(예: `selfdestruct` 사용 시)에 사용할 수 있습니다.
> 구성된 트랜잭션 중 하나라도 되돌려지면 테스트는 실패합니다.

예를 들어, 아래 컨트랙트에서 `testC`는 `testA` 및 `setB(uint256)` 함수에 의해 수정된 상태를 사용하도록 구성되어 있습니다:

```solidity
contract ContractTest is Test {
    uint256 a;
    uint256 b;

    function beforeTestSetup(
        bytes4 testSelector
    ) public pure returns (bytes[] memory beforeTestCalldata) {
        if (testSelector == this.testC.selector) {
            beforeTestCalldata = new bytes[](2);
            beforeTestCalldata[0] = abi.encodePacked(this.testA.selector);
            beforeTestCalldata[1] = abi.encodeWithSignature("setB(uint256)", 1);
        }
    }

    function testA() public {
        require(a == 0);
        a += 1;
    }

    function setB(uint256 value) public {
        b = value;
    }

    function testC() public {
        assertEq(a, 1);
        assertEq(b, 1);
    }
}
```

### 공유 설정 (Shared setups)

도우미 추상 컨트랙트를 만들고 테스트 컨트랙트에서 상속하여 공유 설정을 사용할 수 있습니다:

```solidity
abstract contract HelperContract {
    address constant IMPORTANT_ADDRESS = 0x543d...;
    SomeContract someContract;
    constructor() {...}
}

contract MyContractTest is Test, HelperContract {
    function setUp() public {
        someContract = new SomeContract(0, IMPORTANT_ADDRESS);
        ...
    }
}

contract MyOtherContractTest is Test, HelperContract {
    function setUp() public {
        someContract = new SomeContract(1000, IMPORTANT_ADDRESS);
        ...
    }
}
```

<br></br>

:::tip
호환되지 않는 Solidity 버전을 가진 컨트랙트를 배포하려면 [`getCode`](/reference/cheatcodes/get-code) 치트코드를 사용하세요.
:::
