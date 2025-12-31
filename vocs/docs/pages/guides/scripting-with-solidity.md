---
description: 시뮬레이션, 브로드캐스팅 및 검증 기능을 갖춘 Solidity 스크립트를 사용하여 선언적으로 컨트랙트를 배포합니다.
---

## Solidity로 스크립팅하기

Solidity 스크립팅은 제한적이고 덜 사용자 친화적인 [`forge create`](/forge/reference/create)를 사용하는 대신, Solidity를 사용하여 선언적으로 컨트랙트를 배포하는 방법입니다.

Solidity 스크립트는 Hardhat과 같은 도구로 작업할 때 작성하는 스크립트와 유사합니다. 차이점은 JavaScript 대신 Solidity로 작성된다는 것과, 드라이 런(dry-run) 기능이 포함된 고급 시뮬레이션을 제공하는 빠른 Foundry EVM 백엔드에서 실행된다는 점입니다.

## `forge script`는 어떻게 작동하나요?

`forge script`는 비동기 방식으로 작동하지 않습니다. 먼저 스크립트의 모든 트랜잭션을 수집한 다음, 그 후에 모두 브로드캐스트합니다. 기본적으로 4단계로 나눌 수 있습니다:

1. 로컬 시뮬레이션 - 컨트랙트 스크립트가 로컬 evm에서 실행됩니다. rpc/fork url이 제공되면 해당 컨텍스트에서 스크립트를 실행합니다. `vm.broadcast` 및/또는 `vm.startBroadcast`에서 발생하는 모든 **외부 호출**(static이 아니고 internal이 아닌)은 리스트에 추가됩니다.
2. 온체인 시뮬레이션 - 선택 사항입니다. rpc/fork url이 제공되면, 이전 단계에서 수집된 모든 트랜잭션을 순차적으로 실행합니다.
3. 브로드캐스팅 - 선택 사항입니다. `--broadcast` 플래그가 제공되고 이전 단계가 성공하면, `1` 단계에서 수집되고 `2` 단계에서 시뮬레이션된 트랜잭션을 브로드캐스트합니다.
4. 검증 - 선택 사항입니다. `--verify` 플래그가 제공되고 API 키가 있으며 이전 단계가 성공하면, 컨트랙트 검증을 시도합니다 (예: etherscan).

:::tip
이전에 실패하거나 시간 초과된 트랜잭션은 `--resume` 플래그를 제공하여 다시 제출할 수 있습니다.
:::

이러한 흐름을 고려할 때, 외부 상태/행위자의 영향을 받을 수 있는 트랜잭션은 `2` 단계에서 시뮬레이션된 것과 다른 결과를 초래할 수 있다는 점(예: 프런트 러닝)을 인지하는 것이 중요합니다.

## `Counter` 프로젝트 설정

Foundry가 제공하는 기본 `Counter` 컨트랙트를 배포해 보겠습니다:

```sh
forge init Counter
```

다음으로 모든 것이 올바른지 확인하기 위해 컨트랙트를 컴파일합니다.

```sh
forge build
```

## `Counter` 컨트랙트 배포

::::steps

### RPC 및 Etherscan API 키 가져오기

`Counter` 컨트랙트를 Sepolia 테스트넷에 배포할 예정이지만, 이를 위해서는 몇 가지 필수 조건이 필요합니다:

1. Sepolia RPC URL 가져오기.

[Chainlist](https://chainlist.org/chain/11155111)에서 RPC URL을 가져오거나 [Alchemy](https://www.alchemy.com/) 또는 [Infura](https://www.infura.io/)와 같은 RPC 공급자를 사용할 수 있습니다.

2. 배포를 위한 일회용 개인 키 가져오기.

```sh
`cast wallet new`
```

```
Successfully created new keypair.
Address:     <PUBLIC KEY>
Private key: <PRIVATE_KEY>
```

3. 개인 키에 자금 입금하기.

다양한 수도꼭지(faucet)에서 Sepolia 테스트넷 ETH를 받으세요:

- [작업 증명 수도꼭지](https://sepolia-faucet.pk910.de/)
- [Alchemy](https://sepoliafaucet.com/)
- [Quicknode](https://faucet.quicknode.com/ethereum/sepolia)
- [Chainstack](https://faucet.chainstack.com/)

일부 수도꼭지는 이더리움 메인넷 잔액을 요구할 수 있습니다.

그런 경우, 제어하고 있는 지갑으로 테스트넷 ETH를 청구한 다음 새로 생성한 배포자 키쌍으로 테스트넷 ETH를 전송하세요.

4. [여기](https://docs.etherscan.io/getting-started/viewing-api-usage-statistics)에서 Sepolia Etherscan API 키를 받으세요.

### `foundry.toml` 구성하기

모든 준비가 되면 `.env` 파일을 생성하고 변수를 추가하세요. Foundry는 프로젝트 디렉토리에 있는 `.env` 파일을 자동으로 로드합니다.

.env 파일은 다음 형식을 따라야 합니다:

```sh
SEPOLIA_RPC_URL=
ETHERSCAN_API_KEY=
```

이제 `foundry.toml` 파일을 편집해야 합니다. 프로젝트 루트에 이미 파일이 하나 있을 것입니다.

파일 끝에 다음 줄을 추가하세요:

```toml
[rpc_endpoints]
sepolia = "${SEPOLIA_RPC_URL}"

[etherscan]
sepolia = { key = "${ETHERSCAN_API_KEY}" }
```

이렇게 하면 Sepolia에 대한 [RPC 별칭](/reference/cheatcodes/rpc)이 생성되고 Etherscan API 키가 로드됩니다.

하지만 이는 `getChain` 메서드에는 영향을 미치지 않습니다.

### 스크립트 작성

다음으로 `script` 폴더로 이동하여 `CounterScript`를 찾으세요.

:::tip
복잡한 배포, 특히 멀티 체인 시나리오의 경우 [스크립트 조정을 위한 구성 컨트랙트](/guides/scripting-with-config) 사용을 고려하세요. TOML 파일을 통한 중앙 집중식 구성 관리를 제공하여 스크립트의 유지 보수성과 재사용성을 높이고, 포크 인스턴스화 및 추적을 위한 기본 지원을 제공합니다.
:::

내용을 다음과 같이 수정하세요:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Script, console} from "forge-std/Script.sol";
import {Counter} from "../src/Counter.sol";

contract CounterScript is Script {
    Counter public counter;

    function setUp() public {}

    function run() public {
        vm.startBroadcast();

        counter = new Counter();

        vm.stopBroadcast();
    }
}
```

이제 코드를 읽어보며 실제 의미와 동작을 파악해 봅시다.

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;
```

스크립트라도 스마트 컨트랙트처럼 작동하지만 배포되지는 않는다는 점을 기억하세요. 따라서 Solidity로 작성된 다른 스마트 컨트랙트와 마찬가지로 `pragma version`을 지정해야 합니다.

```solidity
import {Script, console} from "forge-std/Script.sol";
import {Counter} from "../src/Counter.sol";
```

테스트를 작성할 때 테스트 유틸리티를 얻기 위해 Forge Std를 임포트하는 것처럼, 스크립팅 유틸리티도 제공합니다.

다음 줄은 단순히 `Counter` 컨트랙트를 임포트합니다.

```solidity
contract CounterScript is Script {
```

`CounterScript`라는 컨트랙트를 생성하고 Forge Std의 `Script`를 상속받았습니다.

```solidity
function run() external {
```

기본적으로 스크립트는 엔트리포인트인 `run`이라는 함수를 호출하여 실행됩니다.

이는 `.env` 파일에서 개인 키를 로드합니다. **참고:** `.env` 파일에 개인 키를 노출하고 프로그램에 로드할 때는 주의해야 합니다. 이는 권한이 없는 배포자나 로컬/테스트 설정에만 사용하는 것이 좋습니다. 프로덕션 설정의 경우 Foundry가 지원하는 다양한 [지갑 옵션](/forge/reference/script#wallet-options---raw)을 검토하세요.

```solidity
vm.startBroadcast();
```

이것은 메인 스크립트 컨트랙트에서 수행된 호출 및 컨트랙트 생성을 기록하는 특별한 치트코드입니다. 전달할 전송자의 개인 키는 트랜잭션 서명에 사용되도록 지시합니다. 나중에 이 트랜잭션들을 브로드캐스트하여 `Counter` 컨트랙트를 배포할 것입니다.

```solidity
Counter counter = new Counter();
```

여기서 `Counter` 컨트랙트를 생성했습니다. 이 줄 이전에 `vm.startBroadcast()`를 호출했기 때문에 컨트랙트 생성은 Forge에 의해 기록되며, 앞서 언급했듯이 트랜잭션을 브로드캐스트하여 온체인에 컨트랙트를 배포할 수 있습니다. 브로드캐스트 트랜잭션 로그는 기본적으로 `broadcast` 디렉토리에 저장됩니다. `foundry.toml` 파일에서 [`broadcast`](/config/reference/project#broadcast)를 설정하여 로그 위치를 변경할 수 있습니다.

브로드캐스팅 전송자(sender)는 다음 순서대로 확인하여 결정됩니다:

1. `--sender` 인수가 제공된 경우 해당 주소가 사용됩니다.
2. 서명자(예: 개인 키, 하드웨어 지갑, 키스토어)가 정확히 하나만 설정된 경우 해당 서명자가 사용됩니다.
3. 그렇지 않으면 기본 Foundry 전송자(`0x1804c8AB1F12E6bbf3894d4083f33e07309d1f38`) 사용을 시도합니다.

이제 스크립트 스마트 컨트랙트가 무엇을 하는지 알게 되었으니 실행해 봅시다.

### 테스트넷에 배포하기

다음 부분을 진행하려면 앞서 언급한 변수들을 `.env`에 추가했어야 합니다.

프로젝트 루트에서 다음을 실행하세요:

```sh
# .env 파일의 변수 로드
source .env

# 컨트랙트 배포 및 검증
forge script --chain sepolia script/Counter.s.sol:CounterScript --rpc-url $SEPOLIA_RPC_URL --broadcast --verify -vvvv --interactives 1
```

`--interactives 1`에 주목하세요. 이는 개인 키를 입력할 수 있는 대화형 프롬프트를 엽니다. 개발 환경에서의 단순한 테스트넷 배포가 아닌 경우, **강력하게** [하드웨어 지갑이나 암호로 보호된 키스토어를 사용](/guides/best-practices/key-management)할 것을 권장합니다.

```
Enter private key: <PRIVATE_KEY>
```

Forge가 스크립트를 실행하고 트랜잭션을 브로드캐스트합니다. Forge가 트랜잭션 영수증도 기다리므로 시간이 조금 걸릴 수 있습니다. 1분 정도 후에 다음과 같은 내용을 볼 수 있습니다:

```
[⠊] Compiling...
No files changed, compilation skipped
Enter private key:
Traces:
  [137029] CounterScript::run()
    ├─ [0] VM::startBroadcast()
    │   └─ ← [Return]
    ├─ [96345] → new Counter@<ADDRESS>
    │   └─ ← [Return] 481 bytes of code
    ├─ [0] VM::stopBroadcast()
    │   └─ ← [Return]
    └─ ← [Stop]


Script ran successfully.

## Setting up 1 EVM.
==========================
Simulated On-chain Traces:

  [96345] → new Counter@<ADDRESS>
    └─ ← [Return] 481 bytes of code


==========================

Chain 11155111

Estimated gas price: <GAS_PRICE> gwei

Estimated total gas used for script: <GAS>

Estimated amount required: <GAS_AMOUNT> ETH

==========================

##### sepolia
✅  [Success] Hash: <HASH>
Contract Address: <ADDRESS>
Block: <BLOCK>
Paid: <GAS>

✅ Sequence #1 on sepolia | Total Paid: <GAS>


==========================

ONCHAIN EXECUTION COMPLETE & SUCCESSFUL.
##
Start verification for (1) contracts
Start verifying contract `<ADDRESS>` deployed on sepolia
Compiler version: 0.8.28

Submitting verification for [src/Counter.sol:Counter] <ADDRESS>.
Submitted contract for verification:
	Response: `OK`
	GUID: `<GUID>`
	URL: https://sepolia.etherscan.io/address/<ADDRESS>
Contract verification status:
Response: `NOTOK`
Details: `Pending in queue`
Warning: Verification is still pending...; waiting 15 seconds before trying again (7 tries remaining)
Contract verification status:
Response: `OK`
Details: `Pass - Verified`
Contract successfully verified
All (1) contracts were verified!

Transactions saved to: /home/user/counter/broadcast/Counter.s.sol/11155111/run-latest.json

Sensitive values saved to: /home/user/counter/cache/Counter.s.sol/11155111/run-latest.json
```

이것으로 `Counter` 컨트랙트가 Sepolia 테스트넷에 성공적으로 배포되었으며 Etherscan에서 검증까지 완료되었음을 단 하나의 명령어로 확인했습니다.

### 로컬 Anvil 인스턴스에 배포하기

`--fork-url`을 구성하여 로컬 테스트넷인 Anvil에 배포할 수 있습니다.

터미널 창 하나에서 Anvil을 시작해 봅시다:

```sh
anvil
```

그러면 기본 계정 목록이 표시됩니다.

```
Available Accounts
==================

(0) 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 (10000.000000000000000000 ETH)
...

Private Keys
==================

(0) 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
...
```

그런 다음 다른 터미널 창에서 다음 스크립트를 실행하세요:

```sh
forge script script/Counter.s.sol:CounterScript --fork-url http://localhost:8545 --broadcast --interactives 1
```

다음으로 개인 키를 입력하세요. 목록에서 하나를 고르세요.

```
Enter private key: <PRIVATE_KEY>
```

```
[⠊] Compiling...
No files changed, compilation skipped
Enter private key:
Script ran successfully.

## Setting up 1 EVM.

==========================

Chain 31337

Estimated gas price: 2.000000001 gwei

Estimated total gas used for script: 203856

Estimated amount required: 0.000407712000203856 ETH

==========================

##### anvil-hardhat
✅  [Success] Hash: 0x6795deaad7fd483eda4b16af7d8b871c7f6e49beb50709ce1cf0ca81c29247d1
Contract Address: 0x5FbDB2315678afecb367f032d93F642f64180aa3
Block: 1
Paid: 0.000156813000156813 ETH (156813 gas * 1.000000001 gwei)

✅ Sequence #1 on anvil-hardhat | Total Paid: 0.000156813000156813 ETH (156813 gas * avg 1.000000001 gwei)


==========================

ONCHAIN EXECUTION COMPLETE & SUCCESSFUL.

Transactions saved to: /home/user/counter/broadcast/Counter.s.sol/31337/run-latest.json

Sensitive values saved to: /home/user/counter/cache/Counter.s.sol/31337/run-latest.json
```

::::
