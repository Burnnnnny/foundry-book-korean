## anvil

### 이름

anvil - 스마트 컨트랙트 배포 및 테스트를 위한 로컬 테스트넷 노드를 생성합니다. 다른 EVM 호환 네트워크를 포크하는 데에도 사용할 수 있습니다.

### 시놉시스

`anvil` [*options*]

### 설명

스마트 컨트랙트 배포 및 테스트를 위한 로컬 테스트넷 노드를 생성합니다. 다른 EVM 호환 네트워크를 포크하는 데에도 사용할 수 있습니다.

이 섹션에서는 마이닝 모드(Mining Modes), 지원되는 전송 계층(Supported Transport Layers), 지원되는 RPC 메서드(Supported RPC Methods), Anvil 플래그 및 사용법에 대한 광범위한 목록을 다룹니다. 동시에 여러 플래그를 실행할 수 있습니다.

#### 마이닝 모드 (Mining Modes)

마이닝 모드는 Anvil을 사용하여 블록이 얼마나 자주 채굴되는지를 나타냅니다. 기본적으로 트랜잭션이 제출되자마자 자동으로 새 블록을 생성합니다.

원한다면 이 설정을 인터벌 마이닝(interval mining)으로 변경할 수 있습니다. 이는 사용자가 선택한 일정 시간마다 새 블록이 생성됨을 의미합니다. 이러한 유형의 마이닝을 원한다면 다음 예시와 같이 `--block-time <block-time-in-seconds>` 플래그를 추가하여 수행할 수 있습니다.

```sh
# 10초마다 새 블록 생성
anvil --block-time 10
```

`never`라는 세 번째 마이닝 모드도 있습니다. 이 경우 자동 및 인터벌 마이닝을 비활성화하고 대신 필요할 때만 채굴(mine on demand)합니다. 다음과 같이 입력하여 수행할 수 있습니다:

```sh
# never 마이닝 모드 활성화
anvil --no-mining
```

블록의 확정(finalization) 속도를 높이려면 예를 들어 값 `1`과 함께 `--slots-in-an-epoch` 플래그를 사용할 수 있습니다. 이렇게 하면 최신 블록이 `N`일 때 높이 `N-2`의 블록이 확정됩니다.

#### 지원되는 전송 계층 (Supported Transport Layers)

HTTP 및 Websocket 연결이 지원됩니다. 서버는 기본적으로 8545 포트에서 수신 대기하지만 다음 명령을 실행하여 변경할 수 있습니다:

```sh
anvil --port <PORT>
```

#### 기본 CREATE2 배포자 (Default CREATE2 Deployer)

Anvil은 포킹 없이 사용될 때 `0x4e59b44847b379578588920ca78fbf26c0b4956c` 주소에 [기본 CREATE2 배포자 프록시](https://github.com/Arachnid/deterministic-deployment-proxy)를 포함합니다.

이를 통해 포킹 없이 로컬에서 CREATE2 배포를 테스트할 수 있습니다.

#### 지원되는 RPC 메서드 (Supported RPC Methods)

##### 표준 메서드 (Standard Methods)

표준 메서드는 [이 참조](https://ethereum.org/en/developers/docs/apis/json-rpc/)를 기반으로 합니다.

- `web3_clientVersion`

- `web3_sha3`

- `eth_chainId`

- `eth_networkId`

- `eth_gasPrice`

- `eth_accounts`

- `eth_blockNumber`

- `eth_getBalance`

- `eth_getStorageAt`

- `eth_getBlockByHash`

- `eth_getBlockByNumber`

- `eth_getTransactionCount`

- `eth_getBlockTransactionCountByHash`

- `eth_getBlockTransactionCountByNumber`

- `eth_getUncleCountByBlockHash`

- `eth_getUncleCountByBlockNumber`

- `eth_getCode`

- `eth_sign`

- `eth_signTypedData_v4`

- `eth_sendTransaction`

- `eth_sendRawTransaction`

- `eth_call`

- `eth_createAccessList`

- `eth_estimateGas`

- `eth_getTransactionByHash`

- `eth_getTransactionByBlockHashAndIndex`

- `eth_getTransactionByBlockNumberAndIndex`

- `eth_getTransactionReceipt`

- `eth_getUncleByBlockHashAndIndex`

- `eth_getUncleByBlockNumberAndIndex`

- `eth_getLogs`

- `eth_newFilter`

- `eth_getFilterChanges`

- `eth_newBlockFilter`

- `eth_newPendingTransactionFilter`

- `eth_getFilterLogs`

- `eth_uninstallFilter`

- `eth_getWork`

- `eth_subscribe`

- `eth_unsubscribe`

- `eth_syncing`

- `eth_submitWork`

- `eth_submitHashrate`

- `eth_feeHistory`

- `eth_getProof`

- `eth_fillTransaction`

- `debug_traceTransaction`
  `anvil --steps-tracing`을 사용하여 `structLogs`를 얻습니다.

- `debug_traceCall`
  비표준 트레이스는 아직 지원되지 않습니다. 즉, `trace` 매개변수에 어떤 인자도 전달할 수 없습니다.

- `trace_transaction`

- `trace_block`

##### 커스텀 메서드 (Custom Methods)

`anvil_*` 네임스페이스는 `hardhat`의 별칭입니다. 자세한 내용은 [Hardhat 문서](https://hardhat.org/hardhat-network/reference#hardhat-network-methods)를 참조하세요.

`anvil_impersonateAccount`
외부 소유 계정(EOA) 또는 컨트랙트를 가장하여 트랜잭션을 보냅니다.

`anvil_stopImpersonatingAccount`
`anvil_impersonateAccount`로 설정된 경우 계정 또는 컨트랙트 가장을 중지합니다.

`anvil_autoImpersonateAccount`
계정 자동 가장을 활성화하려면 `true`, 비활성화하려면 `false`를 받습니다. 활성화된 경우 모든 트랜잭션의 발신자가 자동으로 가장됩니다. `anvil_impersonateAccount`와 동일합니다.

`anvil_getAutomine`
자동 채굴이 활성화되어 있으면 true, 그렇지 않으면 false를 반환합니다.

`anvil_getBlobByHash`
주어진 KZG commitment 버전 해시에 대한 blob을 반환합니다.

`anvil_getBlobsByTransactionHash`
주어진 트랜잭션 해시에 대한 blob들을 반환합니다.

`anvil_getBlobSidecarsByBlockId`
주어진 블록 ID에 대한 blob sidecar들을 반환합니다.

`anvil_getBlobsByBlockId`
주어진 블록 ID에 대한 blob들을 반환합니다. 선택적으로 버전 해시 목록으로 필터링할 수 있습니다.

`anvil_mine`
일련의 블록을 채굴합니다.

`anvil_dropTransaction`
풀에서 트랜잭션을 제거합니다.

`anvil_reset`
포크를 새로운 포크 상태로 재설정하고 선택적으로 포크 설정을 업데이트합니다.

`anvil_setRpcUrl`
백엔드 RPC URL을 설정합니다.

`anvil_setBalance`
계정의 잔액을 수정합니다.

`anvil_setCode`
컨트랙트의 코드를 설정합니다.

`anvil_setNonce`
주소의 논스(nonce)를 설정합니다.

`anvil_setStorageAt`
계정 스토리지의 단일 슬롯을 씁니다.

`anvil_setCoinbase`
코인베이스(coinbase) 주소를 설정합니다.

`anvil_setLoggingEnabled`
로깅을 활성화하거나 비활성화합니다.

`anvil_setMinGasPrice`
노드의 최소 가스 가격을 설정합니다.

`anvil_setNextBlockBaseFeePerGas`
다음 블록의 기본 수수료(base fee)를 설정합니다.

`anvil_setChainId`
현재 EVM 인스턴스의 체인 ID를 설정합니다.

`anvil_dumpState`
체인의 전체 상태를 나타내는 16진수 문자열을 반환합니다. 동일한 상태를 다시 얻기 위해 Anvil의 새 인스턴스나 재시작된 인스턴스로 다시 가져올 수 있습니다.

`anvil_loadState`
이전에 `anvil_dumpState`에서 반환된 16진수 문자열이 주어지면 내용을 현재 체인 상태에 병합합니다. 충돌하는 계정/스토리지 슬롯을 덮어씁니다.

`anvil_nodeInfo`
현재 실행 중인 Anvil 노드의 설정 매개변수를 검색합니다.

##### 특별 메서드 (Special Methods)

특별 메서드는 Ganache에서 왔습니다. [여기](https://github.com/trufflesuite/ganache-cli-archive/blob/master/README.md)에서 문서를 확인할 수 있습니다.

`evm_setAutomine`
단일 부울(boolean) 인수를 기반으로 네트워크에 제출된 각 새 트랜잭션에 대한 새 블록의 자동 채굴을 활성화하거나 비활성화합니다. `false`로 설정하면 Anvil은 사용자가 설정한 모든 블록 간격 시간마다 새 블록을 계속 채굴합니다. `true`로 설정하면 Anvil은 새 트랜잭션이 감지될 때까지 새 블록 채굴을 일시 중지합니다. 일시 중지 상태에서 주기적 채굴을 재개하려면 evm_setIntervalMining을 호출하여 블록 간격 시간을 설정하세요.

`evm_setIntervalMining`
채굴 동작을 주어진 간격(초)의 인터벌(interval)로 설정합니다.

`evm_snapshot`
현재 블록에서 블록체인의 상태를 스냅샷합니다.

`evm_revert`
블록체인의 상태를 이전 스냅샷으로 되돌립니다. 되돌릴 스냅샷 ID인 단일 매개변수를 받습니다.

`evm_increaseTime`
주어진 시간(초)만큼 시간을 앞으로 이동합니다.

`evm_setNextBlockTimestamp`
`evm_increaseTime`과 유사하지만 다음 블록에 원하는 정확한 타임스탬프를 받습니다.

`anvil_setBlockTimestampInterval`
`evm_increaseTime`과 유사하지만 블록 타임스탬프 `interval`을 설정합니다. 다음 블록의 타임스탬프는 `lastBlock_timestamp + interval`로 계산됩니다.

`evm_setBlockGasLimit`
다음 블록들의 블록 가스 제한을 설정합니다.

`anvil_removeBlockTimestampInterval`
`anvil_setBlockTimestampInterval`이 존재하는 경우 제거합니다.

`evm_mine`
단일 블록을 채굴합니다.

`anvil_enableTraces`
사용자가 트랜잭션을 실행할 때 반환되는 트랜잭션에 대한 호출 추적(call traces)을 켭니다(txhash/receipt 대신).

`eth_sendUnsignedTransaction`
서명 상태에 관계없이 트랜잭션을 실행합니다.

다음 세 가지 메서드에 대해서는 [Geth 문서](https://geth.ethereum.org/docs/rpc/ns-txpool)를 읽어보세요.

`txpool_status`
다음 블록(들)에 포함되기 위해 현재 대기 중인 트랜잭션 수와 향후 실행을 위해 예약된 트랜잭션 수를 반환합니다.

`txpool_inspect`
다음 블록(들)에 포함되기 위해 현재 대기 중인 모든 트랜잭션과 향후 실행을 위해 예약된 트랜잭션의 요약을 반환합니다.

`txpool_content`
다음 블록(들)에 포함되기 위해 현재 대기 중인 모든 트랜잭션과 향후 실행을 위해 예약된 트랜잭션의 세부 정보를 반환합니다.

##### Otterscan 메서드

`ots_*` 네임스페이스는 [Otterscan 사양](https://docs.otterscan.io/api-docs/ots-api)을 구현합니다.

`ots_getApiLevel`
Otterscan이 호환되는 노드에 연결되어 있는지 확인하고 그렇지 않은 경우 친숙한 메시지를 표시하는 데 사용됩니다.

`ots_getInternalOperations`
트랜잭션 내부의 내부 ETH 전송을 반환합니다.

`ots_hasCode`
특정 주소에 배포된 코드가 포함되어 있는지 확인합니다.

`ots_getTransactionError`
트랜잭션 원시(raw) 오류 출력을 추출합니다.

`ots_traceTransaction`
호출, 컨트랙트 생성 및 자체 파괴(self-destructs)의 모든 변형을 추출하고 호출 트리(call tree)를 반환합니다.

`ots_getBlockDetails`
Otterscan의 블록 세부 정보 페이지를 위해 맞춤 제작되고 확장된 eth_getBlock* 버전입니다.

`ots_getBlockTransactions`
특정 블록에 대한 페이지네이션된 트랜잭션을 가져오고 로그와 같은 일부 장황한 필드를 제거합니다.

`ots_searchTransactionsBefore`
특정 주소에 대해 주어진 대상 블록 이전의 페이지네이션된 인바운드/아웃바운드 트랜잭션 호출을 가져옵니다.

`ots_searchTransactionsAfter`
특정 주소에 대해 주어진 대상 블록 이후의 페이지네이션된 인바운드/아웃바운드 트랜잭션 호출을 가져옵니다.

`ots_getTransactionBySenderAndNonce`
주어진 논스(nonce)에 대해 특정 발신자 주소의 트랜잭션 해시를 가져옵니다.

`ots_getContractCreator`
트랜잭션 해시와 컨트랙트를 생성한 주소를 가져옵니다.

#### 지원되는 비콘 노드 API 엔드포인트

JSON-RPC API 외에도 Anvil은 [Eth Beacon Node API](https://ethereum.github.io/beacon-APIs/) 사양의 일부 엔드포인트를 지원합니다.

[`GET /eth/v1/beacon/genesis`](https://ethereum.github.io/beacon-APIs/#/Beacon/getGenesis) 체인의 제네시스(genesis) 세부 정보를 검색합니다. 응답에는 `genesis_time`, `genesis_validators_root`, `genesis_fork_version` 세 가지 필드가 포함됩니다. 그러나 `genesis_time`만 지원되며 나머지 두 필드는 기본적으로 0입니다.


[`GET /eth/v1/beacon/blobs/{block_id}[?versioned_hashes=...]`](https://ethereum.github.io/beacon-APIs/#/Beacon/getBlobs)
주어진 블록 ID에 대한 blob들을 검색합니다. 선택적으로 버전 해시 목록으로 필터링할 수 있습니다.

### 옵션 (OPTIONS)

#### 일반 옵션 (General Options)

`-a, --accounts <ACCOUNTS>`
&nbsp;&nbsp;&nbsp;&nbsp; 계정 수를 설정합니다. [default: 10]

`--auto-impersonate`
&nbsp;&nbsp;&nbsp;&nbsp; 시작 시 자동 가장(autoImpersonate)을 활성화합니다.

`-b, --block-time <block-time>`
&nbsp;&nbsp;&nbsp;&nbsp; 인터벌 마이닝을 위한 블록 시간(초)입니다.

`--balance <BALANCE>`
&nbsp;&nbsp;&nbsp;&nbsp; 계정의 잔액을 설정합니다. [default: 10000]

`--derivation-path <DERIVATION_PATH>`
&nbsp;&nbsp;&nbsp;&nbsp; 파생될 자식 키의 파생 경로를 설정합니다. [default: m/44'/60'/0'/0/]

`-h, --help`
&nbsp;&nbsp;&nbsp;&nbsp; 도움말 정보를 출력합니다.

`--hardfork <HARDFORK>`
&nbsp;&nbsp;&nbsp;&nbsp; 사용할 EVM 하드포크를 선택합니다. 예: `prague`, `cancun`, `shanghai`, `paris`, `london` 등... [default: latest]

`--init <PATH>`
&nbsp;&nbsp;&nbsp;&nbsp; 주어진 `genesis.json` 파일로 제네시스 블록을 초기화합니다.

`-m, --mnemonic <MNEMONIC>`
&nbsp;&nbsp;&nbsp;&nbsp; 계정 생성에 사용되는 BIP39 니모닉 구문입니다.

`--no-mining`
&nbsp;&nbsp;&nbsp;&nbsp; 자동 및 인터벌 마이닝을 비활성화하고 대신 필요할 때 채굴합니다.

`--order <ORDER>`
&nbsp;&nbsp;&nbsp;&nbsp; 멤풀(mempool)에서 트랜잭션이 정렬되는 방식입니다. [default: fees]

`-p, --port <PORT>`
&nbsp;&nbsp;&nbsp;&nbsp; 수신 대기할 포트 번호입니다. [default: 8545]

`--steps-tracing`
&nbsp;&nbsp;&nbsp;&nbsp; geth 스타일 트레이스를 반환하는 디버그 호출에 사용되는 단계 추적(steps tracing)을 활성화합니다. [aliases: tracing]

`--ipc [<PATH>]`
&nbsp;&nbsp;&nbsp;&nbsp; 주어진 `PATH` 인수 또는 기본 경로(unix: `tmp/anvil.ipc`, windows: `\\.\pipe\anvil.ipc`)에서 IPC 엔드포인트를 시작합니다.

`--silent`
&nbsp;&nbsp;&nbsp;&nbsp; 시작 시 아무것도 출력하지 않습니다.

`--timestamp <TIMESTAMP>`
&nbsp;&nbsp;&nbsp;&nbsp; 제네시스 블록의 타임스탬프를 설정합니다.

`-V, --version`
&nbsp;&nbsp;&nbsp;&nbsp; 버전 정보를 출력합니다.

`--disable-default-create2-deployer`
&nbsp;&nbsp;&nbsp;&nbsp; 포킹 없이 Anvil을 실행할 때 기본 CREATE2 팩토리 배포를 비활성화합니다.

#### EVM 옵션 (EVM Options)

`-f, --fork-url <URL>`
&nbsp;&nbsp;&nbsp;&nbsp; 빈 상태에서 시작하는 대신 원격 엔드포인트를 통해 상태를 가져옵니다.

`--fork-block-number <BLOCK>`
&nbsp;&nbsp;&nbsp;&nbsp; 원격 엔드포인트를 통해 특정 블록 번호에서 상태를 가져옵니다 (동일한 명령줄에서 `--fork-url`을 전달해야 함).

`--fork-chain-id <CHAIN>`
&nbsp;&nbsp;&nbsp;&nbsp; 원격 엔드포인트에서 가져오는 것을 건너뛰기 위해 체인 ID를 지정합니다. 이는 오프라인 시작 모드를 활성화합니다.
`--fork-url`과 `--fork-block-number`를 모두 전달해야 하며, 필요한 상태가 이미 디스크에 캐시되어 있어야 합니다. 로컬에 없는 것은 원격에서 가져옵니다.

`--fork-retry-backoff <BACKOFF>`
&nbsp;&nbsp;&nbsp;&nbsp; 오류 발생 시 초기 재시도 백오프(backoff)입니다.

`--fork-transaction-hash <TRANSACTION>`
&nbsp;&nbsp;&nbsp;&nbsp; 원격 엔드포인트를 통해 특정 트랜잭션 해시에서 상태를 가져옵니다 (동일한 명령줄에서 `--fork-url`을 전달해야 함).

`--retries <retries>`
&nbsp;&nbsp;&nbsp;&nbsp; 불안정한 네트워크(요청 시간 초과)에 대한 재시도 요청 수입니다. [default: 5]

`--timeout <timeout>`
&nbsp;&nbsp;&nbsp;&nbsp; 포킹 모드에서 원격 JSON-RPC 서버로 전송된 요청에 대한 시간 초과(ms)입니다. [default: 45000]

`--compute-units-per-second <CUPS>`
&nbsp;&nbsp;&nbsp;&nbsp; 이 공급자에 대해 가정된 초당 사용 가능한 컴퓨팅 단위(compute units) 수를 설정합니다. [default: 330]

`--no-rate-limit`
&nbsp;&nbsp;&nbsp;&nbsp; 이 노드 공급자에 대한 속도 제한을 비활성화합니다. 항상 존재하는 경우 `--compute-units-per-second`를 재정의합니다. [default: false]

`--no-storage-caching`
&nbsp;&nbsp;&nbsp;&nbsp; RPC 캐싱을 비활성화합니다. 모든 스토리지 슬롯을 엔드포인트에서 읽습니다. 이 플래그는 프로젝트의 구성 파일을 재정의합니다 (동일한 명령줄에서 --fork-url을 전달해야 함).

#### 실행자 환경 설정 (Executor Environment Config)

`--base-fee <FEE>`
`--block-base-fee-per-gas <FEE>`
&nbsp;&nbsp;&nbsp;&nbsp; 블록의 기본 수수료(base fee)입니다.

`--chain-id <CHAIN_ID>`
&nbsp;&nbsp;&nbsp;&nbsp; 체인 ID입니다. [default: 31337]

`--code-size-limit <CODE_SIZE>`
&nbsp;&nbsp;&nbsp;&nbsp; EIP-170: 바이트 단위의 컨트랙트 코드 크기 제한입니다. 테스트를 위해 늘리는 데 유용합니다. [default: 0x6000 (~25kb)]

`--gas-limit <GAS_LIMIT>`
&nbsp;&nbsp;&nbsp;&nbsp; 블록 가스 제한입니다.

`--gas-price <GAS_PRICE>`
&nbsp;&nbsp;&nbsp;&nbsp; 가스 가격입니다.

#### 서버 옵션 (Server Options)

`--allow-origin <allow-origin>`
&nbsp;&nbsp;&nbsp;&nbsp; CORS `allow_origin`을 설정합니다. [default: *]

`--no-cors`
&nbsp;&nbsp;&nbsp;&nbsp; CORS를 비활성화합니다.

`--host <HOST>`
&nbsp;&nbsp;&nbsp;&nbsp; 서버가 수신 대기할 IP 주소입니다.

`--config-out <OUT_FILE>`
&nbsp;&nbsp;&nbsp;&nbsp; `anvil`의 출력을 json으로 사용자 지정 파일에 씁니다.

`--prune-history`
&nbsp;&nbsp;&nbsp;&nbsp; 전체 체인 기록을 유지하지 않습니다.

`--no-request-size-limit`
&nbsp;&nbsp;&nbsp;&nbsp; 요청 크기 제한을 비활성화합니다. 기본값은 2MB입니다.

### 예시 (EXAMPLES)

1. 계정 수를 15개로 설정하고 잔액을 300 ETH로 설정

```sh
anvil --accounts 15 --balance 300
```

2. 테스트를 실행할 주소 선택

```sh
anvil --sender 0xC8479C45EE87E0B437c09d3b8FE8ED14ccDa825E
```

3. 멤풀에서 트랜잭션이 정렬되는 방식을 FIFO로 변경

```sh
anvil --order fifo
```

### 쉘 자동 완성 (Shell Completions)

`anvil completions` _shell_

주어진 쉘에 대한 쉘 자동 완성 스크립트를 생성합니다.

지원되는 쉘:

- bash
- elvish
- fish
- powershell
- zsh

#### 예시

1. zsh용 쉘 자동 완성 스크립트 생성:
   ```sh
   anvil completions zsh > $HOME/.oh-my-zsh/completions/_anvil
   ```

### Docker 내 사용 (Usage within Docker)

엔트리포인트 명령에 인수를 전달할 수 없는 [Docker 컨테이너](/guides/foundry-in-docker)를 사용하여 Github Actions에서 서비스를 실행하려면 `ANVIL_IP_ADDR` 환경 변수를 사용하여 호스트의 IP를 설정하세요. `ANVIL_IP_ADDR=0.0.0.0`은 `--host <ip>` 옵션을 제공하는 것과 동일합니다.

#### `genesis.json` 사용하기

Anvil의 `genesis.json` 파일은 Geth와 유사한 목적을 수행하며, 네트워크의 초기 상태, 합의 규칙 및 사전 할당된 계정을 정의하여 모든 노드가 일관되게 시작하고 네트워크 무결성을 유지하도록 합니다. 잔액, 가스 제한 등 모든 값은 16진수로 정의해야 합니다.

- `chainId`: 블록체인의 식별자로, 각 네트워크마다 고유합니다.
- `nonce`: 데이터 무결성을 보장하기 위해 해싱 알고리즘에 사용되는 카운터입니다.
- `timestamp`: 유닉스 시간으로 표시된 제네시스 블록의 생성 시간입니다.
- `extraData`: 제네시스 블록 생성자가 포함할 수 있는 추가 데이터입니다.
- `gasLimit`: 블록에서 사용할 수 있는 최대 가스 양입니다.
- `difficulty`: 새 블록을 채굴하는 것이 얼마나 어려운지에 대한 척도입니다.
- `mixHash`: 블록에 대해 충분한 양의 계산을 증명하는 고유 식별자입니다.
- `coinbase`: 이 블록을 채굴한 채굴자의 이더리움 주소입니다.
- `stateRoot`: 모든 트랜잭션 후의 최종 상태를 반영하는 상태 트라이(state trie)의 루트입니다.
- `alloc`: 사전 정의된 잔액으로 주소 집합에 이더를 사전 할당할 수 있습니다.
- `number`: 블록 번호이며, 제네시스 블록은 0입니다.
- `gasUsed`: 블록에서 사용된 총 가스입니다.
- `parentHash`: 부모 블록의 해시이며, 제네시스 블록은 부모가 없으므로 모두 0입니다.

제네시스를 통한 메인넷 시뮬레이션 샘플은 [여기](https://github.com/paradigmxyz/reth/blob/8f3e4a15738d8174d41f4aede5570ecead141a77/crates/primitives/res/genesis/mainnet.json)에서 찾을 수 있습니다.

```json
{
  "chainId": "0x2323",
  "nonce": "0x42",
  "timestamp": "0x0",
  "extraData": "0x11bbe8db4e347b4e8c937c1c8370e4b5ed33adb3db69cbdb7a38e1e50b1b82fa",
  "gasLimit": "0x1388",
  "difficulty": "0x400000000",
  "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x0000000000000000000000000000000000000000",
  "stateRoot": "0xd7f8974fb5ac78d9ac099b9ad5018bedc2ce0a72dad1827a1709da30580f0544",
  "alloc": {
    "000d836201318ec6899a67540690382780743280": {
      "balance": "0xad78ebc5ac6200000"
    }
  },
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```
