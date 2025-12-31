---
description: Chisel은 자세한 피드백과 프로젝트 통합 기능을 갖춘 대화형 프로토타이핑 및 디버깅을 위한 빠른 Solidity REPL입니다.
---

## Chisel

Chisel은 빠르고 실용적이며 상세한 Solidity REPL입니다.

`chisel` 바이너리는 Foundry 프로젝트 내부와 외부 모두에서 사용할 수 있습니다.
바이너리가 Foundry 프로젝트 루트에서 실행되면 Chisel은 프로젝트의 구성 옵션을 상속합니다.

Chisel은 Foundry 제품군의 일부이며 `forge`, `cast`, `anvil`과 함께 설치됩니다. 아직 Foundry를 설치하지 않았다면 [Foundry 설치](/introduction/installation)를 참조하세요.

### 시작하기

Chisel을 사용하려면 `chisel`을 입력하기만 하면 됩니다.

```sh
chisel
```

여기서부터 Solidity 코드를 작성해 보세요! Chisel은 각 입력에 대해 상세한 피드백을 제공합니다.

변수 `a`를 생성하고 조회합니다:

```console
➜ uint256 a = 123;
➜ a
Type: uint256
├ Hex: 0x7b
├ Hex (full word): 0x000000000000000000000000000000000000000000000000000000000000007b
└ Decimal: 123
```

마지막으로 `!source`를 실행하여 `a`가 적용되었는지 확인합니다:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.28;

import {Vm} from "forge-std/Vm.sol";

contract REPL {
    Vm internal constant vm = Vm(address(uint160(uint256(keccak256("hevm cheat code")))));

    /// @notice REPL contract entry point
    function run() public {
        uint256 a = 123;
    }
}
```

사용 가능한 명령어를 보려면 REPL 내에서 `!help`를 입력하세요.

<br></br>

:::info
Chisel과 그 기능에 대한 자세한 정보는 [`chisel` 참조](/chisel/reference)를 확인하세요.
:::
