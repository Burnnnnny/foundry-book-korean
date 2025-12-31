# 린팅 (Linting)

`forge lint`는 프로젝트의 Solidity 소스 파일을 분석하여 잠재적인 문제를 식별하고 코딩 표준을 강제하는 명령어입니다. 이는 코드베이스 전반에 걸쳐 코드 품질과 일관성을 유지하는 데 도움이 됩니다.

## 예제

1. 프로젝트의 모든 Solidity 파일 린트:

   ```sh
   forge lint
   ```

2. 특정 디렉토리의 파일만 린트:

   ```sh
   forge lint src/contracts/
   ```

3. `high` 및 `gas` 심각도 린트만 사용하여 린트:

   ```sh
   forge lint --severity high --severity gas
   ```

4. 특정 린트 ID로 린트하고 JSON으로 출력:
   ```sh
   forge lint --only-lint incorrect-shift --json
   ```

## 지원되는 린트

이 섹션에서는 `forge lint`에서 지원하는 린트에 대해 자세히 설명합니다. 각 린트에는 ID, 확인하는 문제에 대한 설명, 심각도, 그리고 올바르지 않은 코드와 올바른 코드의 예제가 포함되어 있습니다.

### 높은 심각도 (High Severity)

#### `incorrect-shift`

피연산자가 관습적이지 않거나 잠재적으로 잘못된 순서로 되어 있는 시프트 연산, 특히 리터럴이 비(非)리터럴에 의해 시프트될 때 경고합니다.

Solidity에서 비트 시프트 연산(`<<` 왼쪽 시프트, `>>` 오른쪽 시프트)은 시프트할 값을 왼쪽 피연산자로, 시프트할 비트 수를 오른쪽 피연산자로 취합니다.

이 린트 규칙은 휴리스틱을 사용하여 잠재적으로 잘못된 시프트 연산을 플래그합니다. 이를 위해 리터럴이 변수에 의해 시프트되는 표현식을 식별하는데, 이는 종종 피연산자가 반대로 의도되었을 수 있는 논리적 오류의 징후일 수 있습니다. 만약 그것이 정말 의도된 것이라면, 리터럴을 상수로 대체하는 것이 좋습니다. 또는 린트 규칙을 비활성화할 수 있습니다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract IncorrectShift {
    uint64 const LARGE_NUM = 1 ether;
    uint256 foo = 100;

    function correct() public view returns (uint256) {
        // 'foo'를 리터럴 '2'만큼 시프트.
        return foo << 2;

        // 상수 'LARGE_NUM'을 변수 'foo'만큼 시프트.
        return LARGE_NUM << foo;
    }

    function incorrect() public view returns (uint256) {
        // 리터럴 '2'를 변수 'foo'만큼 시프트. 이는 오류일 가능성이 높습니다.
        return 2 << foo;
    }
}
```

프로젝트에서 이 린트를 비활성화하려면 `foundry.toml` 구성 파일의 `[lint]` 섹션 내 `exclude_lints` 배열에 해당 ID를 추가하면 됩니다:

```toml
[lint]
# ... 나머지 린트 구성 ...
exclude_lints = ["incorrect-shift"]
```

또는 [인라인 구성](/config/reference/linter#inline-configuration)을 사용하여 개별 발생을 비활성화할 수도 있습니다.

#### `unchecked-call`

저수준 호출(`.call()`, `.delegatecall()`, `.staticcall()`)이 성공 반환 값을 확인하지 않을 때 경고합니다.

Solidity의 저수준 호출은 튜플 `(bool success, bytes memory data)`를 반환합니다. `success` 값을 확인하지 않으면 호출된 함수가 되돌려졌음에도(revert) 실행이 계속되는 등 조용한 실패로 이어질 수 있으며, 이는 예기치 않은 동작이나 보안 취약점을 초래할 수 있습니다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract UncheckedCall {
    function correct() public {
        (bool success, ) = address(target).call("");
        require(success, "Call failed");

        // 또는 if문 사용
        (bool ok, bytes memory data) = address(target).call(abi.encodeWithSignature("foo()"));
        if (!ok) revert("Call failed");
    }

    function incorrect() public {
        // 확인되지 않은 호출 - 성공 값이 무시됨
        address(target).call("");

        // 확인되지 않은 호출 - 데이터만 사용됨
        (, bytes memory data) = address(target).call("");
    }
}
```

프로젝트에서 이 린트를 비활성화하려면 `foundry.toml` 구성 파일의 `[lint]` 섹션 내 `exclude_lints` 배열에 해당 ID를 추가하면 됩니다:

```toml
[lint]
# ... 나머지 린트 구성 ...
exclude_lints = ["unchecked-call"]
```

또는 [인라인 구성](/config/reference/linter#inline-configuration)을 사용하여 개별 발생을 비활성화할 수도 있습니다.

#### `erc20-unchecked-transfer`

ERC20 `transfer` 또는 `transferFrom` 호출이 반환 값을 확인하지 않을 때 경고합니다.

ERC20 표준은 이러한 함수가 성공 여부를 나타내는 부울을 반환해야 한다고 명시하고 있지만, 모든 구현이 이 패턴을 올바르게 따르는 것은 아닙니다. 일부 토큰은 실패 시 되돌려지지만(revert), 다른 토큰은 false를 반환합니다. 반환 값을 확인하지 않으면 전송이 조용히 실패하여 자금 손실이나 잘못된 컨트랙트 상태로 이어질 수 있습니다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ERC20UncheckedTransfer {
    IERC20 public token;

    function correct() public {
        // require로 반환 값 확인
        require(token.transfer(recipient, amount), "Transfer failed");

        // 또는 명시적으로 캡처하고 확인
        bool success = token.transferFrom(sender, recipient, amount);
        if (!success) revert("Transfer failed");
    }

    function incorrect() public {
        // 확인되지 않은 전송 - 반환 값이 무시됨
        token.transfer(recipient, amount);

        // 확인되지 않은 transferFrom - 반환 값이 무시됨
        token.transferFrom(sender, recipient, amount);
    }
}
```

프로젝트에서 이 린트를 비활성화하려면 `foundry.toml` 구성 파일의 `[lint]` 섹션 내 `exclude_lints` 배열에 해당 ID를 추가하면 됩니다:

```toml
[lint]
# ... 나머지 린트 구성 ...
exclude_lints = ["erc20-unchecked-transfer"]
```

이 린트는 함수 이름만 확인하고 호출된 주소가 ERC20 컨트랙트인지 확인하지 않기 때문에 오탐(false positive)이 발생할 수 있습니다. 따라서 [인라인 구성](/config/reference/linter#inline-configuration)을 사용하여 개별 발생을 비활성화해야 할 수도 있습니다.

### 중간 심각도 (Medium Severity)

#### divide-before-multiply

같은 표현식 내에서 곱셈 전에 나눗셈을 수행하는 경우, 특히 정수 연산에서 경고합니다.

Solidity에서 정수 나눗셈은 버림(0을 향해 내림)을 수행합니다.
곱셈 전에 나눗셈을 수행하면 의도하지 않은 정밀도 손실이 발생할 수 있으며, 이는 연산 순서를 변경하여 피할 수 있었습니다.
예를 들어 `(a * c) / b`가 0이 아닌 결과를 산출하더라도, `a < b`인 경우 `(a / b) * c`는 `0`이 될 수 있습니다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract DivideBeforeMultiply {
    function correct() public pure returns (uint256) {
        return (1 * 3) / 2; // 결과는 1.
    }

    function incorrect() public pure returns (uint256) {
        return (1 / 2) * 3; // 결과는 0.
    }
}
```

프로젝트에서 이 린트를 비활성화하려면 `foundry.toml` 구성 파일의 `[lint]` 섹션 내 `exclude_lints` 배열에 해당 ID를 추가하면 됩니다:

```toml
[lint]
# ... 나머지 린트 구성 ...
exclude_lints = ["divide-before-multiply"]
```

또는 [인라인 구성](/config/reference/linter#inline-configuration)을 사용하여 개별 발생을 비활성화할 수도 있습니다.

#### `unsafe-typecast`

데이터 손실이나 예기치 않은 동작을 초래할 수 있는 안전하지 않은 형 변환에 대해 경고합니다.

Solidity에서 형 변환은 확인되지 않으며(unchecked) 크기가 다른 타입 간에 변환할 때 예기치 않은 동작을 유발할 수 있습니다.
예를 들어 더 큰 타입에서 더 작은 타입으로 캐스팅(예: `uint256`에서 `uint8`로)하면 상위 비트가 조용히 잘립니다.
이로 인해 데이터 손실이 발생하고 미묘하고 감지하기 어려운 버그가 발생할 수 있습니다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract UnsafeTypecast {
    function safe(uint256 largeValue) public pure {
        if (largeValue > type(uint8).max) revert();
        // 위에서 'largeValue'가 적합함을 확인했으므로 'uint8'로 캐스팅하는 것은 안전합니다.
        // forge-lint: disable-next-line(unsafe-typecast)
        uint8 smallValue = uint8(largeValue);
    }

    function unsafe(uint256 largeValue) public pure {
        // 이 캐스트는 안전하지 않습니다: 8비트에 맞추기 위해 `largeValue`를 자릅니다.
        // 하위 8비트를 제외한 모든 것을 버립니다(즉, `largeValue % 256`).
        // 255보다 큰 값은 데이터가 손실되어 버그로 이어질 수 있습니다.
        uint8 truncated = uint8(largeValue);
    }
}
```

프로젝트에서 이 린트를 비활성화하려면 `foundry.toml` 구성 파일의 `[lint]` 섹션 내 `exclude_lints` 배열에 해당 ID를 추가하면 됩니다:

```toml
[lint]
# ... 나머지 린트 구성 ...
exclude_lints = ["unsafe-typecast"]
```

또는 [인라인 구성](/config/reference/linter#inline-configuration)을 사용하여 개별 발생을 비활성화할 수도 있습니다.

### 정보 / 스타일 가이드 (Informational / Style Guide)

#### `pascal-case-struct`

구조체 이름이 `PascalCase`(예: `MyStruct`) 규칙을 따르도록 합니다. 이는 코드 가독성을 높이고 일관성을 유지하기 위한 Solidity의 일반적인 스타일 가이드라인입니다.

유용한 리소스: [Solidity 스타일 가이드 - 구조체 이름](https://docs.soliditylang.org/en/latest/style-guide.html#struct-names)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract PascalCaseStruct {
    // 맞음
    struct MyStruct {
        uint256 data;
    }

    // 틀림
    struct my_struct {
        uint256 data;
    }
    struct myStruct {
        uint256 data;
    }
}
```

프로젝트에서 이 린트를 비활성화하려면 `foundry.toml` 구성 파일의 `[lint]` 섹션 내 `exclude_lints` 배열에 해당 ID를 추가하면 됩니다:

```toml
[lint]
# ... 나머지 린트 구성 ...
exclude_lints = ["pascal-case-struct"]
```

또는 [인라인 구성](/config/reference/linter#inline-configuration)을 사용하여 개별 발생을 비활성화할 수도 있습니다.

#### `mixed-case-function`

함수 이름이 `mixedCase`(예: `myFunction`) 규칙을 따르도록 합니다. 이는 함수를 구조체나 이벤트와 같은 다른 식별자와 구별하는 데 도움이 되며 표준 관행입니다.

유용한 리소스: [Solidity 스타일 가이드 - 함수 이름](https://docs.soliditylang.org/en/latest/style-guide.html#function-names)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MixedCaseFunction {
    // 맞음
    function myFunction() public pure returns (uint256) {
        return 1;
    }

    // 틀림
    function MyFunction() public pure returns (uint256) {
        return 1;
    }
    function my_function() public pure returns (uint256) {
        return 1;
    }
}
```

프로젝트에서 이 린트를 비활성화하려면 `foundry.toml` 구성 파일의 `[lint]` 섹션 내 `exclude_lints` 배열에 해당 ID를 추가하면 됩니다:

```toml
[lint]
# ... 나머지 린트 구성 ...
exclude_lints = ["mixed-case-function"]
```

또는 [인라인 구성](/config/reference/linter#inline-configuration)을 사용하여 개별 발생을 비활성화할 수도 있습니다.

#### `mixed-case-variable`

변경 가능한 변수 이름(로컬 변수 및 `constant`나 `immutable`이 아닌 상태 변수)이 `mixedCase`(예: `myVariable`) 규칙을 따르도록 합니다. 이는 변수 명명에 대한 일반적인 Solidity 스타일과 일치합니다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MixedCaseVariable {
    // 맞음
    uint256 stateVariable;

    function correct() public {
        uint256 localVariable = 10;
    }

    // 틀림
    uint256 state_variable;

    function incorrect() public {
        uint256 local_variable = 20;
    }
}
```

프로젝트에서 이 린트를 비활성화하려면 `foundry.toml` 구성 파일의 `[lint]` 섹션 내 `exclude_lints` 배열에 해당 ID를 추가하면 됩니다:

```toml
[lint]
# ... 나머지 린트 구성 ...
exclude_lints = ["mixed-case-variable"]
```

또는 [인라인 구성](/config/reference/linter#inline-configuration)을 사용하여 개별 발생을 비활성화할 수도 있습니다.

#### `screaming-snake-case-const`

`constant` 변수 이름이 `SCREAMING_SNAKE_CASE`(예: `MY_CONSTANT`) 규칙을 따르도록 합니다. 이는 Solidity의 상수 표준 규칙으로, 쉽게 식별할 수 있게 해줍니다.

유용한 리소스: [Solidity 스타일 가이드 - 상수](https://docs.soliditylang.org/en/latest/style-guide.html#constants)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ScreamingSnakeCaseConstant {
    // 맞음
    uint256 constant MY_CONSTANT = 1;

    // 틀림
    uint256 constant myConstant = 2;
    uint256 constant my_constant = 3;
}
```

프로젝트에서 이 린트를 비활성화하려면 `foundry.toml` 구성 파일의 `[lint]` 섹션 내 `exclude_lints` 배열에 해당 ID를 추가하면 됩니다:

```toml
[lint]
# ... 나머지 린트 구성 ...
exclude_lints = ["screaming-snake-case-constant"]
```

또는 [인라인 구성](/config/reference/linter#inline-configuration)을 사용하여 개별 발생을 비활성화할 수도 있습니다.

#### `screaming-snake-case-immutable`

`immutable` 변수 이름이 `SCREAMING_SNAKE_CASE`(예: `MY_IMMUTABLE_VAR`) 규칙을 따르도록 합니다. 상수와 마찬가지로 이 규칙은 불변 변수를 구별하고 일관성을 유지하는 데 도움이 됩니다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ScreamingSnakeCaseImmutable {
    // 맞음
    uint256 immutable MY_IMMUTABLE_VAR;

    // 틀림
    uint256 immutable myImmutableVar;
    address immutable my_immutable_var;
}
```

프로젝트에서 이 린트를 비활성화하려면 `foundry.toml` 구성 파일의 `[lint]` 섹션 내 `exclude_lints` 배열에 해당 ID를 추가하면 됩니다:

```toml
[lint]
# ... 나머지 린트 구성 ...
exclude_lints = ["screaming-snake-case-immutable"]
```

또는 [인라인 구성](/config/reference/linter#inline-configuration)을 사용하여 개별 발생을 비활성화할 수도 있습니다.

#### `unused-import`

임포트된 심볼이 소스 파일의 어디에서도 사용되지 않을 때 경고합니다. 사용되지 않는 임포트는 배포 비용을 불필요하게 증가시키고 코드의 명확성을 떨어뜨립니다.

이 린트는 다음을 확인합니다:
- 사용되지 않는 명명된 임포트: `import {Symbol} from "file.sol";`
- 사용되지 않는 별칭 임포트: `import "file.sol" as Alias;`
- 사용되지 않는 별칭 명명된 임포트: `import {Symbol as Alias} from "file.sol";`

> 참고: 별칭이 없는 일반 임포트(`import "file.sol";`)는 부작용을 위해 사용될 수 있으므로 이 린트에서 확인하지 않습니다.

프로젝트에서 이 린트를 비활성화하려면 `foundry.toml` 구성 파일의 `[lint]` 섹션 내 `exclude_lints` 배열에 해당 ID를 추가하면 됩니다:

```toml
[lint]
# ... 나머지 린트 구성 ...
exclude_lints = ["unused-import"]
```

또는 [인라인 구성](/config/reference/linter#inline-configuration)을 사용하여 개별 발생을 비활성화할 수도 있습니다.

#### `unaliased-plain-import`

별칭 없이 일반 임포트를 사용할 때 경고합니다. `import "file.sol";`과 같은 일반 임포트는 심볼이 어디서 오는지 불분명하게 만들고 이름 충돌을 일으킬 수 있습니다.

모범 사례는 다음 중 하나입니다:
- 명명된 임포트 사용: `import {Symbol1, Symbol2} from "file.sol";`
- 별칭 임포트 사용: `import "file.sol" as FileAlias;`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// 맞음 - 명명된 임포트 사용
import {SafeMath, Math} from "./Math.sol";

// 맞음 - 별칭 임포트 사용
import "./Utils.sol" as Utils;

// 틀림 - 별칭 없는 일반 임포트
import "./Helpers.sol";
```

프로젝트에서 이 린트를 비활성화하려면 `foundry.toml` 구성 파일의 `[lint]` 섹션 내 `exclude_lints` 배열에 해당 ID를 추가하면 됩니다:

```toml
[lint]
# ... 나머지 린트 구성 ...
exclude_lints = ["unaliased-plain-import"]
```

또는 [인라인 구성](/config/reference/linter#inline-configuration)을 사용하여 개별 발생을 비활성화할 수도 있습니다.

### 가스 최적화 (Gas Optimizations)

#### `asm-keccak256`

가능한 경우 `keccak256` 해싱에 인라인 어셈블리를 사용할 것을 권장합니다. Solidity 전역 함수 `keccak256()`은 메모리 할당 및 복사를 포함하는데, 이는 특히 작고 고정된 크기 데이터의 해싱을 위해 메모리에서 직접 작동하는 인라인 어셈블리 구현보다 가스 효율성이 떨어질 수 있습니다.

프로덕션 사용의 경우, 일반적인 해싱 패턴에 대해 고도로 최적화된 구현을 제공하는 Solady 라이브러리의 [Vectorized의 EfficientHashLib](https://github.com/Vectorized/solady/blob/main/src/utils/EfficientHashLib.sol) 사용을 고려하세요.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract HashOptimization {
    // 맞음 - 인라인 어셈블리
    function correct(uint256 a, uint256 b) public pure returns (bytes32 hashedVal) {
        assembly {
            mstore(0x00, a)
            mstore(0x20, b)
            let hashedVal := keccak256(0x00, 0x40)
        }
    }

    // 대안 - Solady의 EfficientHashLib 사용
    // import {EfficientHashLib} from "solady/utils/EfficientHashLib.sol";
    // function efficient(uint256 a, uint256 b) public pure returns (bytes32) {
    //     return EfficientHashLib.hash(a, b);
    // }

    // 틀림
    function incorrect(uint256 a, uint256 b) public pure returns (bytes32) {
        return keccak256(abi.encodePacked(a, b));
    }
}
```

프로젝트에서 이 린트를 비활성화하려면 `foundry.toml` 구성 파일의 `[lint]` 섹션 내 `exclude_lints` 배열에 해당 ID를 추가하면 됩니다:

```toml
[lint]
# ... 나머지 린트 구성 ...
exclude_lints = ["asm-keccak256"]
```

또는 [인라인 구성](/config/reference/linter#inline-configuration)을 사용하여 개별 발생을 비활성화할 수도 있습니다.

### 코드 크기 최적화 (Codesize Optimizations)

#### `unwrapped-modifier-logic`

수정자(modifier)에 `_` 자리 표시자 사이에 래핑되지 않은 로직이 포함된 경우 경고합니다. Solidity에서 `_` 앞이나 뒤에 나타나는 수정자의 코드는 해당 수정자를 사용하는 모든 함수에 인라인되므로, 수정자가 여러 번 사용될 때 컨트랙트 크기가 크게 증가할 수 있습니다.

복잡한 로직을 수정자에서 호출되는 내부 함수로 옮기면, 특히 많은 함수에서 사용되는 수정자의 경우 코드 중복을 피하여 바이트코드 크기를 줄일 수 있습니다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract UnwrappedModifierLogic {
    address owner;

    // 맞음 - 복잡한 로직이 내부 함수에 래핑됨
    modifier onlyOwner() {
        _checkOwner();
        _;
    }

    function _checkOwner(address who) internal view {
        require(who == owner, "Not owner");
        // 추가적인 복잡한 검사...
    }

    // 틀림 - 로직이 수정자에 직접 있어 중복됨
    modifier onlyOwnerIncorrect() {
        require(msg.sender == owner, "Not owner");
        // 이 코드는 이 수정자를 사용하는 모든 함수에 인라인됨
        _;
    }
}
```

프로젝트에서 이 린트를 비활성화하려면 `foundry.toml` 구성 파일의 `[lint]` 섹션 내 `exclude_lints` 배열에 해당 ID를 추가하면 됩니다:

```toml
[lint]
# ... 나머지 린트 구성 ...
exclude_lints = ["unwrapped-modifier-logic"]
```

또는 [인라인 구성](/config/reference/linter#inline-configuration)을 사용하여 개별 발생을 비활성화할 수도 있습니다.

### 함께 보기

[린트 구성](/config/reference/linter)
