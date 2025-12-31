---
description: forge create 또는 자동 검증 지원이 포함된 Solidity 스크립팅을 사용하여 스마트 컨트랙트를 모든 네트워크에 배포합니다.
---

## 배포하기

Forge는 [`forge create`](/forge/reference/create) 명령어를 사용하여 스마트 컨트랙트를 특정 네트워크에 배포할 수 있습니다.

Forge CLI는 한 번에 하나의 컨트랙트만 배포할 수 있습니다.

여러 체인에 걸쳐 여러 스마트 컨트랙트를 한 번에 배포하고 검증하려면, Forge의 [Solidity 스크립팅](/guides/scripting-with-solidity)이 더 효율적인 접근 방식입니다.

컨트랙트를 배포하려면 RPC URL(env: `ETH_RPC_URL`)과 컨트랙트를 배포할 계정의 개인 키를 제공해야 합니다. 또한 `--broadcast` 플래그는 안전 예방 조치로 트랜잭션을 네트워크에 게시하는 역할을 하며 `forge script`의 `--broadcast` 플래그와 동일합니다. `--broadcast` 플래그를 전달하지 않으면 트랜잭션은 드라이 런(dry-run)으로 실행됩니다.

`MyContract`를 네트워크에 배포하려면:

```sh
forge create src/MyContract.sol:MyContract --rpc-url <YOUR_RPC_URL> --private-key <YOUR_PRIVATE_KEY> --broadcast
compiling...
success.
Deployer: 0xa735b3c25f...
Deployed to: 0x4054415432...
Transaction hash: 0x6b4e0ff93a...
```

Solidity 파일에는 여러 컨트랙트가 포함될 수 있습니다. 위의 `:MyContract`는 `src/MyContract.sol` 파일에서 배포할 컨트랙트를 지정합니다.

`--constructor-args` 플래그를 사용하여 생성자에 인수를 전달하세요:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import {ERC20} from "solmate/tokens/ERC20.sol";

contract MyToken is ERC20 {
    constructor(
        string memory name,
        string memory symbol,
        uint8 decimals,
        uint256 initialSupply
    ) ERC20(name, symbol, decimals) {
        _mint(msg.sender, initialSupply);
    }
}
```

또한, 네트워크가 지원되는 경우 `--verify`를 전달하여 Etherscan, Sourcify 또는 Blockscout에서 컨트랙트를 검증하도록 Forge에 지시할 수 있습니다.

```sh
forge create src/MyToken.sol:MyToken --rpc-url <YOUR_RPC_URL> \
    --private-key <YOUR_PRIVATE_KEY> \
    --broadcast \
    --etherscan-api-key <YOUR_ETHERSCAN_API_KEY> \
    --verify \
    --constructor-args "ForgeUSD" "FUSD" 18 1000000000000000000000
```

## 멀티 체인 배포

포킹 치트코드를 사용하면 한 번에 여러 체인에 여러 스마트 컨트랙트를 배포하고 검증할 수 있습니다.

예를 들어, 단일 명령어로 Sepolia 메인넷과 Base Sepolia에 `Counter` 컨트랙트를 배포하려면 RPC 엔드포인트와 검증자를 다음과 같이 구성할 수 있습니다:

```toml
[rpc_endpoints]
sepolia = "${SEPOLIA_URL}"
base-sepolia = "${BASE_SEPOLIA_URL}"

[etherscan]
sepolia = { key = "${ETHERSCAN_API_KEY}" }
base-sepolia = { key = "${ETHERSCAN_API_KEY}" }
```

그리고 `CounterScript` 스크립트를 다음과 같이 생성합니다:

```solidity
contract CounterScript is Script {
    function run() public {
        vm.createSelectFork("sepolia");
        vm.startBroadcast();
        new Counter();
        vm.stopBroadcast();

        vm.createSelectFork("base-sepolia");
        vm.startBroadcast();
        new Counter();
        vm.stopBroadcast();
    }
}
```

실행 시:

```sh
forge script script/CounterScript.s.sol --slow --multi --broadcast --private-key <YOUR_PRIVATE_KEY> --verify
```

스크립트는 Sepolia 메인넷 포크를 생성하고(`vm.createSelectFork("sepolia")`), `Counter` 컨트랙트를 배포 및 검증한 다음, Base Sepolia 체인 배포로 이동합니다(`vm.createSelectFork("base-sepolia")`).

사용 가능한 모든 포킹 치트코드 목록은 [`forking`](/reference/cheatcodes/forking) 문서를 참조하세요.

## 기존 컨트랙트 검증하기

배포 후 탐색기에서 컨트랙트를 자동으로 검증하려면 `forge create`와 함께 `--verify` 플래그를 사용하는 것이 좋습니다.
Etherscan의 경우 [`ETHERSCAN_API_KEY`](https://docs.etherscan.io/getting-started/viewing-api-usage-statistics)가 설정되어 있어야 합니다.

이미 배포된 컨트랙트를 검증하는 경우 계속 읽으세요.

[`forge verify-contract`](/forge/reference/verify-contract) 명령어를 사용하여 Etherscan, Sourcify, oklink 또는 Blockscout에서 컨트랙트를 검증할 수 있습니다.

다음 정보를 제공해야 합니다:

- 컨트랙트 주소
- 컨트랙트 이름 또는 컨트랙트 경로 `<path>:<contractname>`
- Etherscan API 키 (env: `ETHERSCAN_API_KEY`) (Etherscan 또는 BscScan / BaseScan / Polygonscan 등 유사한 탐색기에서 검증하는 경우).

또한 다음 정보를 제공해야 할 수도 있습니다:

- 생성자 인수 (ABI 인코딩된 형식), 있는 경우
- 외부 연결 라이브러리 (`src_file_path:library_name:library_address` 형식), 있는 경우
- 빌드에 사용된 [컴파일러 버전](https://etherscan.io/solcversions) (커밋 버전 접두사의 8자리 16진수 포함, 일반적으로 nightly 빌드가 아님). 지정하지 않으면 자동 감지됩니다.
- 최적화 횟수 (Solidity 최적화가 활성화된 경우). 지정하지 않으면 자동 감지됩니다.
- [체인 ID](https://evm-chainlist.netlify.app/) (컨트랙트가 이더리움 메인넷에 없는 경우)

`MyToken`(위 참조)을 검증하고 싶다고 가정해 봅시다. [최적화 횟수](/config/reference/solidity-compiler#optimizer_runs)를 100만으로 설정하고, v0.8.10으로 컴파일했으며, 위와 같이 Sepolia 테스트넷(체인 ID: 11155111)에 배포했습니다. 검증 시 설정하지 않으면 `--num-of-optimizations` 기본값은 0이지만, 배포 시 설정하지 않으면 기본값은 200이므로 기본 컴파일 설정을 그대로 둔 경우 `--num-of-optimizations 200`을 전달해야 합니다.

검증 방법은 다음과 같습니다:

```bash
forge verify-contract \
    --chain-id 11155111 \
    --num-of-optimizations 1000000 \
    --watch \
    --constructor-args $(cast abi-encode "constructor(string,string,uint256,uint256)" "ForgeUSD" "FUSD" 18 1000000000000000000000) \
    --verifier etherscan \
    --etherscan-api-key <your_etherscan_api_key> \
    --compiler-version v0.8.10+commit.fc410830 \
    <CONTRACT_ADDRESS> \
    src/MyToken.sol:MyToken

Submitted contract for verification:
                Response: `OK`
                GUID: `a6yrbjp5prvakia6bqp5qdacczyfhkyi5j1r6qbds1js41ak1a`
                url: https://sepolia.etherscan.io//address/0x6a54…3a4c#code
```

> ℹ️ **참고:**
>
> 외부 라이브러리는 `--libraries` 인수를 사용하여 연결된 각 라이브러리마다 하나씩 지정할 수 있습니다. 예를 들어, 두 개의 연결된 라이브러리(`Maths`와 `Utils`)가 있는 컨트랙트를 검증하려면 `forge verify-contract`를 다음과 같이 실행해야 합니다.
>
> ```bash
> --libraries src/lib/Maths.sol:Maths:<maths_lib_address> \
> --libraries src/lib/Utils.sol:Utils:<utils_lib_address>
> ```

검증 결과를 폴링하려면 `verify-contract` 명령어와 함께 [`--watch`](/forge/reference/verify-contract#verify-contract-options) 플래그를 사용하는 것이 좋습니다.

`--watch` 플래그를 제공하지 않은 경우, [`forge verify-check`](/forge/reference/verify-check) 명령어로 검증 상태를 확인할 수 있습니다:

```bash
forge verify-check --chain-id 11155111 <GUID> <YOUR_ETHERSCAN_API_KEY>
Contract successfully verified.
```

<br></br>

> 💡 **팁**
>
> 인수를 ABI 인코딩하려면 Cast의 [`abi-encode`](/cast/reference/abi-encode)를 사용하세요.
>
> 이 예제에서는 `cast abi-encode "constructor(string,string,uint8,uint256)" "ForgeUSD" "FUSD" 18 1000000000000000000000`를 실행하여 인수를 ABI 인코딩했습니다.

<br></br>

### 문제 해결

##### `missing hex prefix ("0x") for hex string`

개인 키 문자열이 `0x`로 시작하는지 확인하세요.

##### `EIP-1559 not activated`

RPC 서버에서 EIP-1559가 지원되지 않거나 활성화되지 않았습니다. `--legacy` 플래그를 전달하여 EIP-1559 트랜잭션 대신 레거시 트랜잭션을 사용하세요. 로컬 환경에서 개발하는 경우 Ganache 대신 Hardhat을 사용할 수 있습니다.

##### `Failed to parse tokens`

전달된 인수의 타입이 올바른지 확인하세요.

##### `Signature error`

개인 키가 올바른지 확인하세요.

##### `Compiler version commit for verify`

로컬에서 실행 중인 정확한 커밋을 확인하려면 `~/.svm/0.x.y/solc-0.x.y --version`을 시도해 보세요. 여기서 `x`와 `y`는 각각 메이저 및 마이너 버전 번호입니다. 출력은 다음과 같습니다:

```bash
solc, the solidity compiler commandline interface
Version: 0.8.12+commit.f00d7308.Darwin.appleclang
```

참고: "0.8.12+commit.f00d7308.Darwin.appleclang" 전체 문자열을 컴파일러 버전 인수로 붙여넣을 수 없습니다. 하지만 커밋의 8자리 16진수를 사용하여 [컴파일러 버전](https://etherscan.io/solcversions)에서 복사하여 붙여넣어야 할 정확한 내용을 찾을 수 있습니다.

##### `Invalid API Key`

[Etherscan API V2](https://docs.etherscan.io/etherscan-v2)에서는 Etherscan 키만 유효하며, BscScan/BaseScan/Polygonscan과 같은 유사한 탐색기에도 사용할 수 있습니다. 다른 탐색기의 레거시 키는 더 이상 사용되지 않습니다.

### 알려진 문제

#### 모호한 임포트 경로가 있는 컨트랙트 검증

Forge는 소스 디렉토리(`src`, `lib`, `test` 등)를 `--include-path` 인수로 컴파일러에 전달합니다.
즉, 다음과 같은 프로젝트 트리가 주어지면

```text
|- src
|-- folder
|--- Contract.sol
|--- IContract.sol
```

`Contract.sol` 내부에서 `folder/IContract.sol` 임포트 경로를 사용하여 `IContract`를 임포트할 수 있습니다.

Etherscan은 이러한 소스를 다시 컴파일할 수 없습니다. 상대 임포트 경로를 사용하도록 임포트를 변경하는 것을 고려하세요.
