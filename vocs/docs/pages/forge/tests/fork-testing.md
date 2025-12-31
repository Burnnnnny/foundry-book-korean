---
description: 포크 모드 또는 포크 치트코드를 사용하여 라이브 블록체인 상태에 대해 실제 컨트랙트와의 통합 테스트를 수행합니다.
---

## 포크 테스트 (Fork Testing)

Forge는 두 가지 접근 방식을 통해 포크된 환경에서의 테스트를 지원합니다:

- [**포크 모드 (Forking Mode)**](#forking-mode) — `forge test --fork-url` 플래그를 통해 모든 테스트에 단일 포크를 사용합니다.
- [**포크 치트코드 (Forking Cheatcodes)**](#forking-cheatcodes) — [포크 치트코드](/reference/cheatcodes/forking)를 통해 Solidity 테스트 코드에서 직접 여러 포크를 생성, 선택 및 관리합니다.

어떤 접근 방식을 사용해야 할까요? 포크 모드는 특정 포크 환경에서 전체 테스트 스위트를 실행하는 데 적합하며, 포크 치트코드는 테스트에서 여러 포크를 다룰 때 더 많은 유연성과 표현력을 제공합니다. 특정 사용 사례와 테스트 전략에 따라 어떤 접근 방식을 사용할지 결정하면 됩니다.

:::tip
Foundry v1.3.0부터는 계정 데이터 페치(fetch) 요청을 3개에서 1개로 줄이는 `eth_getAccountInfo` API 덕분에 Reth 클라이언트를 사용하는 포크 테스트 속도가 빨라졌습니다.
:::

### 포크 모드 (Forking Mode)

포크된 이더리움 메인넷과 같은 포크된 환경에서 모든 테스트를 실행하려면 `--fork-url` 플래그를 통해 RPC URL을 전달하세요:

```bash
forge test --fork-url <your_rpc_url>
```

다음 값들은 포크 시점의 체인 값을 반영하도록 변경됩니다:

- [`block_number`](/config/reference/testing#block_number)
- [`chain_id`](/config/reference/testing#chain_id)
- [`gas_limit`](/config/reference/testing#gas_limit)
- [`gas_price`](/config/reference/testing#gas_price)
- [`block_base_fee_per_gas`](/config/reference/testing#block_base_fee_per_gas)
- [`block_coinbase`](/config/reference/testing#block_coinbase)
- [`block_timestamp`](/config/reference/testing#block_timestamp)
- [`block_difficulty`](/config/reference/testing#block_difficulty)

`--fork-block-number`를 사용하여 포크할 블록을 지정할 수 있습니다:

```bash
forge test --fork-url <your_rpc_url> --fork-block-number 1
```

포킹은 기존 컨트랙트와 상호작용해야 할 때 특히 유용합니다. 실제 네트워크에 있는 것처럼 통합 테스트를 수행할 수 있습니다.

#### 캐싱 (Caching)

`--fork-url`과 `--fork-block-number`가 모두 지정되면, 해당 블록의 데이터는 나중에 테스트를 실행할 때 사용할 수 있도록 캐시됩니다.

데이터는 `~/.foundry/cache/rpc/<chain name>/<block number>`에 캐시됩니다. 캐시를 지우려면 단순히 디렉토리를 제거하거나 [`forge clean`](/forge/reference/clean)(모든 빌드 아티팩트 및 캐시 디렉토리 제거)을 실행하면 됩니다.

또한 `--no-storage-caching`을 전달하거나 `foundry.toml`에서 [`no_storage_caching`](/config/reference/testing#no_storage_caching) 및 [`rpc_storage_caching`](/config/reference/testing#rpc_storage_caching)을 구성하여 캐시를 완전히 무시할 수도 있습니다.

#### 개선된 트레이스 (Improved traces)

Forge는 Etherscan을 사용하여 포크된 환경에서 컨트랙트를 식별하는 기능을 지원합니다.

이 기능을 사용하려면 `--etherscan-api-key` 플래그를 통해 Etherscan API 키를 전달하세요:

```bash
forge test --fork-url <your_rpc_url> --etherscan-api-key <your_etherscan_api_key>
```

또는 `ETHERSCAN_API_KEY` 환경 변수를 설정할 수도 있습니다.

### 포크 치트코드 (Forking Cheatcodes)

포크 치트코드를 사용하면 Solidity 테스트 코드에서 프로그래밍 방식으로 포크 모드에 진입할 수 있습니다. `forge` CLI 인수를 통해 포크 모드를 구성하는 대신, 이러한 치트코드를 사용하면 테스트별로 포크 모드를 사용하고 테스트에서 여러 포크를 다룰 수 있습니다. 각 포크는 고유한 `uint256` 식별자로 구별됩니다.

#### 사용법

_모든_ 테스트 함수는 격리되어 있으며, 각 테스트 함수는 `setUp` _이후_ 상태의 _사본_으로 실행되고 고유한 독립형 EVM에서 실행된다는 점을 기억하는 것이 중요합니다.

따라서 `setUp` 중에 생성된 포크는 테스트에서 사용할 수 있습니다. 아래 코드 예제는 [`createFork`](/reference/cheatcodes/create-fork)를 사용하여 두 개의 포크를 생성하지만 처음에는 하나를 선택하지 _않습니다_. 각 포크는 처음 생성될 때 할당되는 고유 식별자(`uint256 forkId`)로 식별됩니다.

특정 포크를 활성화하려면 해당 `forkId`를 [`selectFork`](/reference/cheatcodes/select-fork)에 전달하여 수행합니다.

[`createSelectFork`](/reference/cheatcodes/create-select-fork)는 `createFork`와 `selectFork`를 한 번에 수행하는 함수입니다.

> 경고: `vm.createSelectFork`는 호출될 때마다 **새로운** 포크를 생성합니다. 동일한 RPC URL로 `vm.createSelectFork`를 여러 번 호출하면, 각각 깨끗한 상태에서 시작하는 여러 개의 독립적인 포크가 생성됩니다. 한 포크에서 이루어진 상태 변경은 `vm.createSelectFork`로 생성된 후속 포크에는 존재하지 않습니다. 포크 간 전환 시 상태 변경을 보존하려면, `vm.createFork`를 한 번 사용하여 각 포크를 생성하고 반환된 포크 ID를 저장한 다음 `vm.selectFork`를 사용하여 전환하세요.

한 번에 하나의 포크만 활성화할 수 있으며, 현재 활성화된 포크의 식별자는 [`activeFork`](/reference/cheatcodes/active-fork)를 통해 검색할 수 있습니다.

[`roll`](/reference/cheatcodes/roll)과 유사하게 [`rollFork`](/reference/cheatcodes/roll-fork)를 사용하여 포크의 `block.number`를 설정할 수 있습니다.

포크가 선택될 때 어떤 일이 발생하는지 이해하려면 포크 모드가 일반적으로 어떻게 작동하는지 아는 것이 중요합니다:

각 포크는 독립적인 EVM입니다. 즉, 모든 포크는 완전히 독립적인 스토리지를 사용합니다. 유일한 예외는 포크 교체(swap) 간에 지속되는 `msg.sender`와 테스트 컨트랙트 자체의 상태입니다.
즉, 포크 `A`가 활성화된 동안(`selectFork(A)`) 변경된 모든 내용은 포크 `A`의 스토리지에만 기록되며 다른 포크가 선택되면 사용할 수 없습니다. 그러나 테스트 컨트랙트 자체에 기록된 변경 사항(변수)은 테스트 컨트랙트가 _지속적인(persistent)_ 계정이므로 여전히 사용할 수 있습니다.

`selectFork` 치트코드는 _원격(remote)_ 섹션을 포크의 데이터 소스로 설정하지만, _로컬(local)_ 메모리는 포크 교체 간에 지속됩니다. 이는 또한 `selectFork`가 언제든지 모든 포크와 함께 호출되어 _원격_ 데이터 소스를 설정할 수 있음을 의미합니다. 그러나 `read/write` 액세스에 대한 위의 규칙이 항상 적용된다는 점을 명심해야 합니다. 즉, _쓰기_는 포크 교체 간에 지속됩니다.

#### 예제

##### 포크 생성 및 선택

```solidity
contract ForkTest is Test {
    // 포크 식별자
    uint256 mainnetFork;
    uint256 optimismFork;

    // .env 파일의 변수에 vm.envString("varname")을 통해 접근
    // ALCHEMY_KEY를 알케미 키 또는 이더스캔 키로 교체하고, 필요한 경우 RPC URL 변경
    // .env 파일 내부 예:
    // MAINNET_RPC_URL = 'https://eth-mainnet.g.alchemy.com/v2/ALCHEMY_KEY'
    // string MAINNET_RPC_URL = vm.envString("MAINNET_RPC_URL");
    // string OPTIMISM_RPC_URL = vm.envString("OPTIMISM_RPC_URL");

    // setup 중에 두 개의 _서로 다른_ 포크 생성
    function setUp() public {
        mainnetFork = vm.createFork(MAINNET_RPC_URL);
        optimismFork = vm.createFork(OPTIMISM_RPC_URL);
    }

    // 포크 id가 고유함을 증명
    function testForkIdDiffer() public {
        assert(mainnetFork != optimismFork);
    }

    // 특정 포크 선택
    function testCanSelectFork() public {
        // 포크 선택
        vm.selectFork(mainnetFork);
        assertEq(vm.activeFork(), mainnetFork);

        // 이제부터 EVM이 요청하면 `mainnetFork`에서 데이터를 가져오고 `mainnetFork`의 스토리지에 씁니다.
    }

    // 동일한 테스트에서 여러 포크 관리
    function testCanSwitchForks() public {
        vm.selectFork(mainnetFork);
        assertEq(vm.activeFork(), mainnetFork);

        vm.selectFork(optimismFork);
        assertEq(vm.activeFork(), optimismFork);
    }

    // 포크는 언제든지 생성 가능
    function testCanCreateAndSelectForkInOneStep() public {
        // 새 포크를 생성하고 선택
        uint256 anotherFork = vm.createSelectFork(MAINNET_RPC_URL);
        assertEq(vm.activeFork(), anotherFork);
    }

    // 포크의 `block.number` 설정
    function testCanSetForkBlockNumber() public {
        vm.selectFork(mainnetFork);
        vm.rollFork(1_337_000);

        assertEq(block.number, 1_337_000);
    }
}
```

##### 분리된 스토리지와 지속적인 스토리지

앞서 언급했듯이 각 포크는 본질적으로 분리된 스토리지를 가진 독립적인 EVM입니다.

포크가 선택될 때 `msg.sender`와 테스트 컨트랙트(`ForkTest`)의 계정만 지속됩니다. 그러나 [`makePersistent`](/reference/cheatcodes/make-persistent)를 사용하여 모든 계정을 지속적인 계정으로 전환할 수 있습니다.

_지속적인(persistent)_ 계정은 고유합니다. 즉, 모든 포크에 존재합니다.

```solidity
contract ForkTest is Test {
    // 포크 식별자
    uint256 mainnetFork;
    uint256 optimismFork;

    // .env 파일의 변수에 vm.envString("varname")을 통해 접근
    // ALCHEMY_KEY를 알케미 키 또는 이더스캔 키로 교체하고, 필요한 경우 RPC URL 변경
    // .env 파일 내부 예:
    // MAINNET_RPC_URL = 'https://eth-mainnet.g.alchemy.com/v2/ALCHEMY_KEY'
    // string MAINNET_RPC_URL = vm.envString("MAINNET_RPC_URL");
    // string OPTIMISM_RPC_URL = vm.envString("OPTIMISM_RPC_URL");

    // setup 중에 두 개의 _서로 다른_ 포크 생성
    function setUp() public {
        mainnetFork = vm.createFork(MAINNET_RPC_URL);
        optimismFork = vm.createFork(OPTIMISM_RPC_URL);
    }

    // 포크가 활성화된 동안 새 컨트랙트 생성
    function testCreateContract() public {
        vm.selectFork(mainnetFork);
        assertEq(vm.activeFork(), mainnetFork);

        // 새 컨트랙트는 `mainnetFork`의 스토리지에 기록됨
        SimpleStorageContract simple = new SimpleStorageContract();

        // 정상적으로 사용 가능
        simple.set(100);
        assertEq(simple.value(), 100);

        // 다른 컨트랙트로 전환한 후에도 `address(simple)`은 알 수 있지만 컨트랙트는 `mainnetFork`에만 존재함
        vm.selectFork(optimismFork);

        /* `simple`이 이제 활성 포크에 존재하지 않는 컨트랙트를 가리키므로 이 호출은 리버트됩니다.
        * 다음과 같은 리버트 메시지가 생성됩니다:
        *
        * "Contract 0xCe71065D4017F316EC606Fe4422e11eB2c47c246 does not exist on active fork with id `1`
        *       But exists on non active forks: `[0]`"
        */
        simple.value();
    }

     // 포크가 활성화된 동안 새로운 _지속적인_ 컨트랙트 생성
     function testCreatePersistentContract() public {
        vm.selectFork(mainnetFork);
        SimpleStorageContract simple = new SimpleStorageContract();
        simple.set(100);
        assertEq(simple.value(), 100);

        // 컨트랙트를 지속적인 것으로 표시하여 다른 포크가 활성화될 때도 사용할 수 있게 함
        vm.makePersistent(address(simple));
        assert(vm.isPersistent(address(simple)));

        vm.selectFork(optimismFork);
        assert(vm.isPersistent(address(simple)));

        // 이제 컨트랙트가 `optimismFork`에서도 사용 가능하므로 성공합니다.
        assertEq(simple.value(), 100);
     }
}

contract SimpleStorageContract {
    uint256 public value;

    function set(uint256 _value) public {
        value = _value;
    }
}
```

자세한 내용과 예제는 [포크 치트코드](/reference/cheatcodes/forking) 참조를 확인하세요.

### EVM 버전

다른 EVM 버전을 사용하는 체인에서 포크된 테스트를 실행하려면 적절한 구성이 필요합니다:

- 사용되는 모든 포크된 체인에 동일한 EVM 버전이 적용되는 경우, `foundry.toml` 파일에서 전역적으로 구성할 수 있습니다:

```toml
evm_version = "prague"
```

- 다른 EVM 버전을 사용하는 경우, 인라인 구성을 사용하여 특정 EVM 테스트 버전을 설정할 수 있습니다:

```solidity
/// forge-config: default.evm_version = "shanghai"
```
