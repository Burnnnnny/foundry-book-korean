---
description: Forge 표준 라이브러리는 치트코드, 어설션, 로깅 및 스토리지 조작을 포함한 필수 테스트 유틸리티를 제공합니다.
---

## Forge 표준 라이브러리 (Forge Std) 개요

Forge 표준 라이브러리(줄여서 Forge Std)는 테스트 작성을 더 쉽고 빠르며 사용자 친화적으로 만드는 유용한 컨트랙트 모음입니다.

Forge Std를 사용하는 것은 Foundry로 테스트를 작성하는 가장 권장되는 방법입니다.

테스트 작성을 시작하는 데 필요한 모든 필수 기능을 제공합니다:

- `Vm.sol`: 최신 치트코드 인터페이스
- `console.sol` 및 `console2.sol`: Hardhat 스타일의 로깅 기능
- `Script.sol`: [Solidity로 스크립팅하기](/guides/scripting-with-solidity)를 위한 기본 유틸리티
- `Test.sol`: 표준 라이브러리, 치트코드 인스턴스(`vm`), Hardhat 콘솔을 포함하는 DSTest의 상위 집합

간단히 `Test.sol`을 가져오고 테스트 컨트랙트에서 `Test`를 상속하세요:

```solidity
import {Test} from "forge-std/Test.sol";

contract ContractTest is Test { ...
```

이제 다음을 수행할 수 있습니다:

```solidity
// `vm` 인스턴스를 통해 Hevm에 접근
vm.startPrank(alice);

// Dappsys Test를 사용하여 단언(Assert) 및 로그
assertEq(dai.balanceOf(alice), 10000e18);

// Hardhat `console` (`console2`)로 로그
console.log(alice.balance);

// Forge Std 표준 라이브러리의 모든 기능 사용
deal(address(dai), alice, 10000e18);
```

`Vm` 인터페이스 또는 `console` 라이브러리를 개별적으로 가져오려면:

```solidity
import {Vm} from "forge-std/Vm.sol";
```

```solidity
import {console} from "forge-std/console.sol";
```

**참고:** `console2.sol`은 Forge가 콘솔 호출에 대한 트레이스를 디코딩할 수 있도록 `console.sol`에 대한 패치를 포함하고 있지만, Hardhat과는 호환되지 않습니다.

```solidity
import {console2} from "forge-std/console2.sol";
```

### 표준 라이브러리

Forge Std는 현재 6개의 표준 라이브러리로 구성되어 있습니다.

#### Std Logs

Std Logs는 [`DSTest`](/reference/ds-test#logging) 라이브러리의 로깅 이벤트를 확장합니다.

#### Std Assertions

Std Assertions는 [`DSTest`](/reference/ds-test#asserting) 라이브러리의 어설션 함수를 확장합니다.

#### Std Cheats

Std Cheats는 Forge 치트코드를 래퍼(wrapper)하여 더 안전하게 사용하고 개발자 경험(DX)을 향상시킵니다.

다른 내부 함수와 마찬가지로 테스트 컨트랙트 내에서 간단히 호출하여 Std Cheats에 접근할 수 있습니다:

```solidity
// 100 ETH 잔액을 가진 Alice로 장난(prank) 설정
hoax(alice, 100 ether);
```

#### Std Errors

Std Errors는 일반적인 내부 Solidity 오류 및 리버트에 대한 래퍼를 제공합니다.

Std Errors는 [`expectRevert`](/reference/cheatcodes/expect-revert) 치트코드와 함께 사용할 때 가장 유용합니다. 내부 Solidity 패닉 코드를 직접 기억할 필요가 없기 때문입니다. 라이브러리이므로 `stdError`를 통해 접근해야 한다는 점에 유의하세요.

```solidity
// 다음 호출에서 산술 오류 예상 (예: 언더플로우)
vm.expectRevert(stdError.arithmeticError);
```

#### Std Storage

Std Storage는 컨트랙트 스토리지 조작을 쉽게 만듭니다. 특정 변수와 연관된 스토리지 슬롯을 찾아 쓸 수 있습니다.

`Test` 컨트랙트는 이미 `stdstore`라는 `StdStorage` 인스턴스를 제공하며, 이를 통해 모든 std-storage 기능에 접근할 수 있습니다. 먼저 테스트 컨트랙트에 `using stdStorage for StdStorage`를 추가해야 한다는 점에 유의하세요.

```solidity
// `game` 컨트랙트에서 `score` 변수를 찾아
// 값을 10으로 변경
stdstore
    .target(address(game))
    .sig(game.score.selector)
    .checked_write(10);
```

#### Std Math

Std Math는 Solidity에서 제공하지 않는 유용한 수학 함수가 있는 라이브러리입니다.

라이브러리이므로 `stdMath`를 통해 접근해야 한다는 점에 유의하세요.

```solidity
// -10의 절대값 가져오기
uint256 ten = stdMath.abs(-10)
```

<br></br>

:::info
Forge 표준 라이브러리에 대한 전체 개요는 [Forge 표준 라이브러리 참조](/reference/forge-std/overview)를 확인하세요.
:::
