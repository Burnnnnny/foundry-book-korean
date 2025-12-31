---
description: 이식 가능하고 재현 가능한 개발 워크플로우를 위해 Foundry의 Docker 이미지를 사용하여 스마트 컨트랙트를 빌드, 테스트 및 배포합니다.
---

## Docker 내부에서 Foundry 실행하기

이 가이드는 Foundry의 Docker 이미지를 사용하여 스마트 컨트랙트를 빌드, 테스트 및 배포하는 방법을 보여줍니다. [`first steps`] 가이드의 코드를 각색했습니다. 아직 해당 가이드를 완료하지 않았고 Solidity를 처음 접하는 경우, 먼저 해당 가이드를 시작하는 것이 좋습니다. 또는 Docker와 Solidity에 어느 정도 익숙하다면 기존 프로젝트를 사용하여 그에 맞게 조정할 수 있습니다.

> 이 가이드는 예시 목적으로만 제공되며 있는 그대로 제공됩니다. 이 가이드는 감사를 거치지 않았으며 완전히 테스트되지 않았습니다. 이 가이드의 어떠한 코드도 프로덕션 환경에서 사용해서는 안 됩니다.

### 설치 및 설정

이 가이드를 실행하는 데 필요한 유일한 설치는 Docker이며, 선택적으로 원하는 IDE가 필요합니다.
[Docker 설치 지침](/introduction/installation)을 따르세요.

향후 명령어를 간결하게 유지하기 위해 이미지 태그를 다시 지정해 보겠습니다:
`docker tag ghcr.io/foundry-rs/foundry:latest foundry:latest`

Foundry를 로컬에 설치하는 것이 엄격하게 필수는 아니지만 디버깅에 도움이 될 수 있습니다. [foundryup](/introduction/installation#using-foundryup)을 사용하여 설치할 수 있습니다.

마지막으로, 이 가이드의 `cast` 또는 `forge create` 부분을 사용하려면 이더리움 노드에 액세스해야 합니다. 자체 노드를 실행하고 있지 않다면(그럴 가능성이 높음), 타사 노드 서비스를 사용할 수 있습니다. 이 가이드에서는 특정 공급자를 추천하지 않습니다. 노드 서비스(Nodes-as-a-Service)에 대해 배우기 좋은 곳은 [이더리움의 관련 문서](https://ethereum.org/en/developers/docs/nodes-and-clients/nodes-as-a-service/)입니다.

**이 가이드의 나머지 부분에서는 이더리움 노드의 RPC 엔드포인트가 다음과 같이 설정되어 있다고 가정합니다**: `export RPC_URL=<YOUR_RPC_URL>`

### Foundry 도커 이미지 둘러보기

도커 이미지는 주로 두 가지 방식으로 사용될 수 있습니다:

1. forge와 cast에 직접 인터페이스로 사용
2. 자체 컨테이너화된 테스트, 빌드 및 배포 도구를 구축하기 위한 기본 이미지로 사용

두 가지 모두 다루겠지만, 먼저 docker를 사용하여 foundry와 인터페이스하는 것부터 시작하겠습니다. 이는 로컬 설치가 제대로 되었는지 확인하는 좋은 테스트이기도 합니다!

도커 이미지에 대해 모든 `cast` [명령어](/cast/reference/cast)를 실행할 수 있습니다. 최신 블록 정보를 가져와 봅시다:

```sh
docker run foundry "cast block --rpc-url $RPC_URL latest"
baseFeePerGas        "0xb634241e3"
difficulty           "0x2e482bdf51572b"
extraData            "0x486976656f6e20686b"
gasLimit             "0x1c9c380"
gasUsed              "0x652993"
hash                 "0x181748772da2f968bcc91940c8523bb6218a7d57669ded06648c9a9fb6839db5"
logsBloom            "0x406010046100001198c220108002b606400029444814008210820c04012804131847150080312500300051044208430002008029880029011520380060262400001c538d00440a885a02219d49624aa110000003094500022c003600a00258009610c410323580032000849a0408a81a0a060100022505202280c61880c80020e080244400440404520d210429a0000400010089410c8408162903609c920014028a94019088681018c909980701019201808040004100000080540610a9144d050020220c10a24c01c000002005400400022420140e18100400e10254926144c43a200cc008142080854088100128844003010020c344402386a8c011819408"
miner                "0x1ad91ee08f21be3de0ba2ba6918e714da6b45836"
mixHash              "0xb920857687476c1bcb21557c5f6196762a46038924c5f82dc66300347a1cfc01"
nonce                "0x1ce6929033fbba90"
number               "0xdd3309"
parentHash           "0x39c6e1aa997d18a655c6317131589fd327ae814ef84e784f5eb1ab54b9941212"
receiptsRoot         "0x4724f3b270dcc970f141e493d8dc46aeba6fffe57688210051580ac960fe0037"
sealFields           []
sha3Uncles           "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347"
size                 "0x1d6bb"
stateRoot            "0x0d4b714990132cf0f21801e2931b78454b26aad706fc6dc16b64e04f0c14737a"
timestamp            "0x6246259b"
totalDifficulty      "0x9923da68627095fd2e7"
transactions         [...]
uncles               []
```

Solidity [소스 코드](https://github.com/dmfxyz/foundry-docker-tutorial)가 있는 디렉토리에 있다면, 해당 디렉토리를 Docker에 마운트하고 원하는 대로 `forge`를 사용할 수 있습니다. 예를 들어:

```sh
docker run -v $PWD:/app foundry "forge test --root /app --watch"
```

코드가 컨테이너 내부에서 완전히 컴파일되고 테스트되는 것을 볼 수 있습니다. 또한 `--watch` 옵션을 전달했으므로 컨테이너는 변경 사항이 감지될 때마다 코드를 다시 컴파일합니다.

### "빌드 및 테스트" 이미지 생성

Foundry 도커 이미지를 기본으로 사용하여 자체 Docker 이미지를 만들어 보겠습니다. 이 이미지를 사용하여 다음을 수행합니다:

1. solidity 코드 빌드
2. solidity 테스트 실행

간단한 `Dockerfile`로 이 두 가지 목표를 달성할 수 있습니다:

```docker
# 최신 foundry 이미지 사용
FROM ghcr.io/foundry-rs/foundry

# 소스 코드를 컨테이너에 복사
WORKDIR /app

# 소스 코드 빌드 및 테스트
COPY . .
RUN forge build
RUN forge test
```

이 도커 이미지를 빌드하고 컨테이너 내에서 forge가 테스트를 빌드/실행하는 것을 지켜볼 수 있습니다:

```sh
docker build --no-cache --progress=plain .
```

이제 테스트 중 하나가 실패하면 어떻게 될까요? `src/test/Counter.t.sol`을 수정하여 잘못된 어설션을 만드세요. 이미지를 다시 빌드해 보세요.

```solidity
function testFuzz_SetNumber(uint256 x) public {
    counter.setNumber(x);
    assertEq(counter.number(), 5);
}
```

```sh
docker build --no-cache --progress=plain .
<...>
#9 0.522 Failed tests:
#9 0.522 [FAIL: assertion failed: 425 != 5; counterexample: calldata=[...] args=[425]] testFuzz_SetNumber(uint256) (runs: 0, μ: 0, ~: 0)
#9 0.522
#9 0.522 Suite result: FAILED. 1 passed; 1 failed; 0 skipped; finished in 686.53µs (407.06µs CPU time)
------
error: failed to solve: executor failed running [/bin/sh -c forge test]: exit code: 1
```

테스트가 실패했기 때문에 이미지가 빌드되지 않았습니다! 이는 실제로 좋은 속성입니다. 성공적으로 빌드된(따라서 사용할 수 있는) Docker 이미지가 있다면, 이미지 내부의 코드가 테스트를 통과했음을 알 수 있기 때문입니다.\*

> \*물론, 도커 이미지의 관리 연속성(chain of custody)은 매우 중요합니다. Docker 레이어 해시는 검증에 매우 유용할 수 있습니다. 프로덕션 환경에서는 [도커 이미지 서명](https://docs.docker.com/engine/security/trust/#:~:text=To%20sign%20a%20Docker%20Image,the%20local%20Docker%20trust%20repository)을 고려하세요.

### "배포자(deployer)" 이미지 생성

이제 좀 더 고급 Dockerfile로 넘어가 보겠습니다. 빌드된(그리고 테스트된!) 이미지를 사용하여 코드를 배포할 수 있도록 엔트리포인트를 추가해 보겠습니다. 먼저 Sepolia 테스트넷을 대상으로 할 수 있습니다.

```docker
# 최신 foundry 이미지 사용
FROM ghcr.io/foundry-rs/foundry

# 소스 코드를 컨테이너에 복사
WORKDIR /app

# 소스 코드 빌드 및 테스트
COPY . .
RUN forge build
RUN forge test

# 엔트리포인트를 forge 배포 명령어로 설정
ENTRYPOINT ["forge", "create"]
```

이번에는 이름을 지정하여 이미지를 빌드해 보겠습니다:

```sh
docker build --no-cache --progress=plain -t counter .
```

도커 이미지를 사용하여 배포하는 방법은 다음과 같습니다:

```sh
docker run counter-deployer --rpc-url $RPC_URL --private-key $PRIVATE_KEY ./src/Counter.sol:Counter
No files changed, compilation skipped
Deployer: 0x496e09fcb240c33b8fda3b4b74d81697c03b6b3d
Deployed to: 0x23d465eaa80ad2e5cdb1a2345e4b54edd12560d3
Transaction hash: 0xf88c68c4a03a86b0e7ecb05cae8dea36f2896cd342a6af978cab11101c6224a9
```

방금 Docker 컨테이너 내에서 컨트랙트를 빌드, 테스트 및 배포했습니다! 이 가이드는 테스트넷을 위한 것이었지만, 메인넷을 대상으로 똑같은 Docker 이미지를 실행할 수 있으며 동일한 도구로 동일한 코드가 배포되고 있음을 확신할 수 있습니다.

### 왜 이것이 유용한가요?

Docker는 이식성, 재현성 및 환경 불변성에 관한 것입니다. 즉, 환경, 네트워크, 개발자 등을 전환할 때 예상치 못한 변경 사항에 대해 덜 걱정해도 됩니다. 스마트 컨트랙트 배포에 Docker 이미지를 사용하려는 몇 가지 기본적인 예는 다음과 같습니다:

- 배포 환경 간에 시스템 수준 종속성이 일치하는지 확인하는 오버헤드를 줄입니다 (예: 프로덕션 러너가 개발 러너와 항상 동일한 버전의 `forge`를 가지고 있습니까?)
- 코드가 배포 전에 테스트되었으며 변경되지 않았다는 확신을 높입니다 (예: 위 이미지에서 배포 시 코드가 다시 컴파일되면 주요 위험 신호입니다).
- 업무 분리에 대한 고충을 완화합니다: 메인넷 자격 증명을 가진 사람은 최신 컴파일러, 코드베이스 등을 가지고 있는지 확인할 필요가 없습니다. 누군가 테스트넷에서 실행한 도커 배포 이미지가 메인넷을 대상으로 하는 것과 동일한지 확인하기 쉽습니다.
- Docker는 사실상 모든 퍼블릭 클라우드 제공업체에서 허용되는 표준입니다. 블록체인과 상호 작용해야 하는 작업, 태스크 등을 쉽게 예약할 수 있습니다.

### `docker-compose`를 사용하여 Anvil 실행

[Docker Compose](https://docs.docker.com/compose/)를 사용하여 Anvil을 시작하려면 다음 `docker-compose.yml` 구성을 사용할 수 있습니다:

```yml
services:
  anvil:
    image: ghcr.io/foundry-rs/foundry
    container_name: anvil
    environment:
      ANVIL_IP_ADDR: "0.0.0.0"
    working_dir: /anvil
    ports:
      - "8545:8545"
    command: anvil
```

마지막으로 `docker compose up`을 실행합니다.

```
docker compose up
[+] Running 1/1
 ✔ Container anvil  Created
Attaching to anvil
anvil  |
anvil  |
anvil  |                              _   _
anvil  |                             (_) | |
anvil  |       __ _   _ __   __   __  _  | |
anvil  |      / _` | | '_ \  \ \ / / | | | |
anvil  |     | (_| | | | | |  \ V /  | | | |
anvil  |      \__,_| |_| |_|   \_/   |_| |_|
anvil  |
anvil  |     0.3.1-dev (fea38858b0 2025-01-21T16:48:49.865302749Z)
anvil  |     https://github.com/foundry-rs/foundry
anvil  |     ...
```
