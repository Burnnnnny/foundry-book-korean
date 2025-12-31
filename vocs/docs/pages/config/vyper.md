## Vyper 지원

Foundry는 Vyper 컨트랙트의 컴파일 및 테스트를 지원합니다.

### 1. 컴파일 (Compilation)

Vyper 설치 방법은 [여기](https://vyper.readthedocs.io/en/stable/installing-vyper.html)를 따르세요. PATH에 `vyper`가 있다면 Foundry가 자동으로 사용합니다.

그렇지 않다면, `foundry.toml`에 `vyper` 경로를 다음과 같이 추가할 수 있습니다:
```toml
[vyper]
path = "/path/to/vyper"
```

#### `forge install`을 통한 Vyper 라이브러리

Vyper 컨트랙트에서 다음과 같은 import를 사용하려면:

```vyper
from snekmate.utils import eip712_domain_separator
```

`forge install`을 통해 원하는 Vyper 라이브러리를 설치할 수 있습니다. 예: `forge install pcaversaccio/snekmate`.

그런 다음 `foundry.toml`을 다음과 같이 조정해야 합니다 ("snekmate"를 원하는 패키지 이름으로 대체):

```toml
skip = ["**/lib/snekmate/**"]
libs = ["lib", "lib/snekmate/src"]
```

#### `pip`을 통한 Vyper 라이브러리

만약 `pip`을 통해 시스템의 python 구성이나 가상 환경에 패키지를 설치하고 싶다면, `foundry.toml`을 수정하여 Foundry가 해당 경로를 가리키도록 할 수 있습니다:

```toml
# .venv에 가상 환경이 있고 Python 3.12를 사용한다고 가정
libs = ["lib", ".venv/lib/python3.12/site-packages/"]
```

`uv`와 같은 호환 가능한 대체 파이썬 패키지 관리자도 작동합니다.

### 2. Solidity 테스트

간단한 Counter 컨트랙트에 대한 테스트를 작성해 보겠습니다:

```vyper
number: public(uint256)

@deploy
@payable
def __init__(initial_number: uint256):
    self.number = initial_number

@external
def set_number(new_number: uint256):
    self.number = new_number

@external
def increment():
    self.number += 1
```

`forge-std`의 `deployCode` 칝코드를 사용하여 배포하고 다음 Solidity 테스트로 테스트할 수 있습니다:
```solidity
import {Test} from "forge-std/Test.sol";

interface ICounter {
    function increment() external;
    function number() external view returns (uint256);
    function set_number(uint256 newNumber) external;
}

contract CounterTest is Test {
    ICounter public counter;
    uint256 initialNumber = 5;

    function setUp() public {
        counter = ICounter(deployCode("Counter", abi.encode(initialNumber)));
        assertEq(counter.number(), initialNumber);
    }

    function test_Increment() public {
        counter.increment();
        assertEq(counter.number(), initialNumber + 1);
    }

    function testFuzz_SetNumber(uint256 x) public {
        counter.set_number(x);
        assertEq(counter.number(), x);
    }
}
```

### 3. 배포하기 (Deploying)

`forge create` 명령어로 Vyper 컨트랙트를 배포할 수 있습니다:
```bash
forge create Counter --constructor-args '1' --rpc-url $RPC_URL --private-key $PRIVATE_KEY
```

그리고 스크립트에서도 `deployCode`를 사용하여 Vyper 컨트랙트를 배포할 수 있습니다:
```solidity
import {Script} from "forge-std/Script.sol";

contract CounterScript is Script {
    function run() public {
        vm.broadcast();
        deployCode("src/Counter.vy", abi.encode(1));
    }
}
```

### 4. Vyper 스크립트

Solidity 스크립트와 같은 방식으로 Vyper 스크립트를 작성할 수 있습니다:
```vyper
interface Vm:
    def startBroadcast(): nonpayable

interface ICounter:
    def increment(): nonpayable
    def number() -> uint256: view

vm: constant(Vm) = Vm(0x7109709ECfa91a80626fF3989D68f67F5b1DD12D)

@external
def run(counter: address):
    number_before: uint256 = staticcall ICounter(counter).number()

    extcall vm.startBroadcast()
    extcall ICounter(counter).increment()

    number_after: uint256 = staticcall ICounter(counter).number()

    assert number_after == number_before + 1
```

이러한 스크립트는 다음 명령어로 실행할 수 있습니다:
```bash
forge script script/Increment.s.vy  --sig 'run' '<counter address>' --rpc-url $RPC_URL --broadcast  --private-key $PRIVATE_KEY
```

### 5. 제한 사항 (Limitations)

- Vyper에서 테스트와 스크립트를 작성하고 실행할 수 있지만, Vyper에는 컨트랙트를 배포할 수 있는 `new` 키워드가 없습니다. 이는 향후 새로운 칝코드로 해결될 예정입니다.
- Vyper는 이름은 같지만 매개변수 타입이 다른 오버로드를 허용하지 않습니다. 따라서 일부 칝코드 조합은 우회 방법이 필요할 수 있습니다. (예: `startBroadcast(address sender)`와 `startBroadcast(uint256 pk)`)
- `forge coverage`는 현재 Vyper 컨트랙트를 지원하지 않습니다.
