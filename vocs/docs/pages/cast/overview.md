---
description: Cast는 커맨드 라인에서 이더리움과 상호 작용하기 위한 스위스 아미 나이프입니다 - 트랜잭션 전송, 컨트랙트 호출 및 체인 데이터 검색을 수행합니다.
---

## Cast

Cast는 커맨드 라인에서 이더리움 애플리케이션과 상호 작용하기 위한 스위스 아미 나이프입니다. 스마트 컨트랙트 호출, 트랜잭션 전송 또는 모든 유형의 체인 데이터 검색을 모두 커맨드 라인에서 수행할 수 있습니다!

`cast` 바이너리는 Foundry 프로젝트 안팎에서 모두 사용할 수 있습니다.

Cast는 Foundry 제품군의 일부이며 `forge`, `chisel`, `anvil`과 함께 설치됩니다. 아직 Foundry를 설치하지 않았다면 [Foundry 설치](/introduction/installation)를 참조하세요.

### 시작하기

다음은 몇 가지 사용 예입니다:

**이더리움 메인넷의 최신 블록 확인**:

```sh
cast block-number --rpc-url https://reth-ethereum.ithaca.xyz/rpc
```

**`vitalik.eth`의 이더 잔액 확인**

```sh
cast balance vitalik.eth --ether --rpc-url https://reth-ethereum.ithaca.xyz/rpc
```

**트랜잭션 재생 및 추적**

```sh
cast run 0x9c32042f5e997e27e67f82583839548eb19dc78c4769ad6218657c17f2a5ed31 --rpc-url https://reth-ethereum.ithaca.xyz/rpc
```

검증된 소스 맵을 사용하여 트랜잭션 추적을 디코딩하고 더 상세하고 사람이 읽을 수 있는 정보를 제공하려면 선택적으로 `--etherscan-api-key <API_KEY>`를 전달하세요.

**DAI 토큰의 총 공급량 검색**

```sh
// [!include ~/snippets/output/cast/cast-call:all]
```

**콜 데이터(calldata) 디코딩**

```sh
// [!include ~/snippets/output/cast/cast-4byte-calldata:all]
```

**두 Anvil 계정 간 메시지 전송**

```sh
cast send --private-key <PRIVATE_KEY> 0x3c44cdddb6a900fa2b585dd299e03d12fa4293bc $(cast from-utf8 "hello world") --rpc-url http://127.0.0.1:8545/
```

<br></br>

:::info
사용 가능한 모든 하위 명령어에 대한 전체 개요는 [`cast` 참조](/cast/reference/cast)를 확인하세요.
:::
