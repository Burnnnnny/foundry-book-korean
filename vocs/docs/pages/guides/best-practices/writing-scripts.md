---
description: 적절한 테스트 및 프런트러닝 방지를 통해 안전하고 유지 보수가 쉬운 배포 스크립트를 작성하기 위한 모범 사례입니다.
---

## 스크립팅 관행

### 기본 함수로 `run` 사용

명확성을 위해 기본 함수 이름으로 `run`을 고수하세요.

### 올바른 함수 가시성 할당

스크립트에서 직접 호출하도록 의도되지 않은 모든 메서드는 `internal` 또는 `private`이어야 합니다. 일반적으로 유일한 `public` 메서드는 `run`이어야 합니다. 각 스크립트 파일이 한 가지 작업만 수행할 때 읽고 이해하기 쉽기 때문입니다.

### 스크립트 파일 명명

프로토콜 수명 주기 동안 실행될 순서에 따라 스크립트 접두사에 숫자를 붙이는 것을 고려하세요. 예: `01_Deploy.s.sol`, `02_TransferOwnership.s.sol`. 이렇게 하면 상황이 더 명확해집니다(self-documenting). 프로젝트에 따라 항상 적용되는 것은 아닐 수 있습니다.

### 스크립트 테스트

- 스크립트 실행으로 인한 상태 변경을 어설션(assert)하는 테스트를 작성하여 일반 컨트랙트를 테스트하는 것처럼 단위 테스트하세요.
- 배포 스크립트를 작성하고 해당 스크립트를 실행하여 테스트를 스캐폴딩하세요. 그런 다음 프로덕션 배포 스크립트의 결과 상태에 대해 모든 테스트를 실행하세요. 이는 배포 스크립트에 대한 확신을 얻을 수 있는 좋은 방법입니다.
- 스크립트 자체 내에서 `require` 문(또는 선호하는 경우 `if (condition) revert()` 패턴)을 사용하여 문제가 있을 경우 스크립트 실행을 중지하세요. 예: `require(computedAddress == deployedAddress, "address mismatch")`. 대신 `assertEq` 도우미를 사용하면 실행이 중지되지 않습니다.

### 브로드캐스트 트랜잭션 감사

**어떤 트랜잭션이 브로드캐스트되는지 신중하게 감사하세요**. 브로드캐스트되지 않은 트랜잭션은 여전히 테스트 컨텍스트에서 실행되므로, 누락된 브로드캐스트나 추가 브로드캐스트는 이전 단계에서 오류의 쉬운 원인이 됩니다.

### 프런트러닝(Frontrunning) 방지

**프런트러닝을 주의하세요**. Forge는 스크립트를 시뮬레이션하고 시뮬레이션 결과에서 트랜잭션 데이터를 생성한 다음 트랜잭션을 브로드캐스트합니다. 시뮬레이션과 브로드캐스트 사이의 체인 상태 변경에 대해 스크립트가 견고한지 확인하세요. 이에 취약한 샘플 스크립트는 다음과 같습니다:

```solidity
// 의사 코드, 컴파일되지 않을 수 있음.
contract VulnerableScript is Script {
   function run() public {
      vm.startBroadcast();

      // 트랜잭션 1: CREATE로 새 Gnosis Safe 배포.
      // CREATE2 대신 CREATE를 사용하므로 새 Safe의 주소는
      // gnosisSafeProxyFactory의 논스(nonce)에 대한 함수입니다.
      address mySafe = gnosisSafeProxyFactory.createProxy(singleton, data);

      // 트랜잭션 2: 새 Safe로 토큰 전송.
      // mySafe의 주소는 gnosisSafeProxyFactory의 논스에 대한 함수임을 알고 있습니다.
      // 시뮬레이션과 브로드캐스트 사이에 다른 누군가가 Gnosis Safe를 배포하면,
      // mySafe의 주소가 달라지고 이 스크립트는 다른 사람의 Safe로 1000 DAI를 전송하게 됩니다.
      // 이 경우 CREATE 대신 CREATE2를 사용하여 이 문제를 방지할 수 있지만,
      // 모든 상황에는 다른 해결책이 있을 수 있습니다.
      dai.transfer(mySafe, 1000e18);

      vm.stopBroadcast();
   }
}
```

### JSON 파일 읽기

JSON 입력 파일에서 읽는 스크립트의 경우 입력 파일을 `script/input/<chainID>/<description>.json`에 두세요. 그런 다음 `run(string memory input)`(또는 여러 파일에서 읽어야 하는 경우 여러 문자열 입력)을 스크립트 서명으로 사용하고 아래 메서드를 사용하여 JSON 파일을 읽으세요.

```solidity
function readInput(string memory input) internal returns (string memory) {
  string memory inputDir = string.concat(vm.projectRoot(), "/script/input/");
  string memory chainDir = string.concat(vm.toString(block.chainid), "/");
  string memory file = string.concat(input, ".json");
  return vm.readFile(string.concat(inputDir, chainDir, file));
}
```
