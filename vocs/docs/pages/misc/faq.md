---
description: 일반적인 Foundry 설치 및 사용 문제에 대한 자주 묻는 질문과 문제 해결 가이드입니다.
---

## FAQ

자주 묻는 질문과 답변 모음입니다. 여기에 질문이 없다면 [Telegram 지원 채널][tg-support]에 들어와서 도움을 요청하세요!

### 소스에서 빌드할 수 없습니다!

최신 안정 버전의 Rust 툴체인을 사용하고 있는지 확인하세요:

```sh
rustup default stable
rustup update stable
```

### `forge`/`cast` 실행 시 `libusb` 오류 발생

릴리스된 바이너리를 사용하는 경우 MacOS에서 다음과 같은 오류가 표시될 수 있습니다:

```sh
dyld: Library not loaded: /usr/local/opt/libusb/lib/libusb-1.0.0.dylib
```

이 문제를 해결하려면 `libusb` 라이브러리를 설치해야 합니다:

```sh
brew install libusb
```

### 오래된 `GLIBC`

`foundryup` 사용 후 다음과 유사한 오류가 발생하는 경우:

```sh
forge: /lib/x86_64-linux-gnu/libc.so.6: version 'GLIBC_2.29' not found (required by forge)
```

2가지 해결 방법이 있습니다:

1. [소스에서 빌드](/introduction/installation/#building-from-source)
2. [Docker 사용](/introduction/installation/#using-foundry-with-docker)

### 도와주세요! 로그를 볼 수 없습니다!

Forge는 기본적으로 로그를 표시하지 않습니다. Hardhat의 `console.log` 또는 DSTest 스타일의 `log_*` 이벤트에서 로그를 보려면,
[`forge test`][forge-test]를 상세도(verbosity) 2(`-vv`)로 실행해야 합니다.

컨트랙트가 방출하는 다른 이벤트를 보려면 트레이스를 활성화하여 실행해야 합니다.
그렇게 하려면 상세도를 3(`-vvv`)으로 설정하여 실패한 테스트에 대한 트레이스를 보거나, 4(`-vvvv`)로 설정하여 모든 테스트에 대한 트레이스를 확인하세요.

### 테스트가 실패하는데 이유를 모르겠습니다!

테스트 실패 원인에 대한 더 나은 통찰력을 얻으려면 트레이스를 사용해 보세요. 트레이스를 활성화하려면 [forge test][forge-test]의 상세도를 적어도 3(`-vvv`)으로 높여야 하지만, 더 많은 트레이스를 위해 5(`-vvvvv`)까지 올릴 수 있습니다.

[트레이스 이해하기][traces] 장에서 트레이스에 대해 자세히 알아볼 수 있습니다.

### `console.log`는 어떻게 사용하나요?

Hardhat의 `console.log`를 사용하려면 [여기][console-log]에서 파일을 복사하여 프로젝트에 추가해야 합니다.

또는 `console.log`가 번들로 제공되는 [Forge Std][forge-std]를 사용할 수 있습니다. Forge Std에서 `console.log`를 사용하려면
다음과 같이 임포트해야 합니다:

```solidity
import {console} from "forge-std/console.sol";
```

### 특정 테스트만 실행하려면 어떻게 해야 하나요?

몇 가지 테스트만 실행하려면 [`forge test`][forge-test]에서 `--match-test`를 사용하여 테스트 함수를 필터링하고,
`--match-contract`를 사용하여 테스트 컨트랙트를 필터링하며, `--match-path`를 사용하여 테스트 파일을 필터링할 수 있습니다.

### 특정 Solidity 컴파일러는 어떻게 사용하나요?

Forge는 프로젝트에 맞는 Solidity 컴파일러를 자동으로 감지하려고 시도합니다.

특정 Solidity 컴파일러를 사용하려면 [설정 파일][config]에서 [`solc`][config-solc]를 설정하거나,
이를 지원하는 Forge 명령(예: [`forge build`][forge-build] 또는 [`forge test`][forge-test])에 `--use solc:<version>`을 전달할 수 있습니다.
solc 바이너리 경로도 허용됩니다. 특정 로컬 solc 바이너리를 사용하려면 설정 파일에서 `solc = "<path to solc>"`를 설정하거나 `--use "<path to solc>"`를 전달할 수 있습니다.
solc 버전/경로는 환경 변수 `FOUNDRY_SOLC=<version/path>`를 통해서도 설정할 수 있지만, cli 인수 `--use`가 우선순위를 가집니다.

예를 들어, 모든 0.7.x Solidity 버전을 지원하는 프로젝트가 있지만 solc 0.7.0으로 컴파일하고 싶다면 `forge build --use solc:0.7.0`을 사용할 수 있습니다.

### 라이브 네트워크에서 포크하려면 어떻게 해야 하나요?

라이브 네트워크에서 포크하려면 [`forge test`][forge-test]에 `--fork-url <URL>`을 전달하세요.
또한 `--fork-block-number <BLOCK>`을 사용하여 특정 블록에서 포크할 수 있는데, 이는 테스트에 결정론(determinism)을 추가하고 Forge가 해당 블록의 체인 데이터를 캐시할 수 있게 합니다.

예를 들어, 블록 10,000,000에서 이더리움 메인넷을 포크하려면 다음을 사용할 수 있습니다: `forge test --fork-url $MAINNET_RPC_URL --fork-block-number 10000000`.

### 나만의 어설션(assertion)을 추가하려면 어떻게 해야 하나요?

자신만의 기본 테스트 컨트랙트를 만들고 선택한 테스트 프레임워크를 상속받아 나만의 어설션을 추가할 수 있습니다.

예를 들어, DSTest를 사용하는 경우 다음과 같이 기본 테스트 컨트랙트를 만들 수 있습니다:

```solidity
contract TestBase is DSTest {
    function myCustomAssertion(uint a, uint b) {
      if (a != b) {
          emit log_string("a and b did not match");
          fail();
      }
    }
}
```

그러면 테스트 컨트랙트에서 `TestBase`를 상속받습니다.

```solidity
contract MyContractTest is TestBase {
    function testSomething() {
        // ...
    }
}
```

마찬가지로 [Forge Std][forge-std]를 사용하는 경우, `Test`를 상속받는 기본 테스트 컨트랙트를 만들 수 있습니다.

헬퍼 메서드와 사용자 정의 어설션이 있는 기본 테스트 컨트랙트의 좋은 예시는 [Solmate의 `DSTestPlus`][dstestplus]를 참조하세요.

### Forge를 오프라인으로 사용하려면 어떻게 해야 하나요?

Forge는 때때로 프로젝트에 맞는 새로운 Solidity 버전을 확인합니다. Forge를 오프라인으로 사용하려면 `--offline` 플래그를 사용하세요.

### Solc 오류가 발생합니다

[solc-bin](https://binaries.soliditylang.org/)은 Apple 실리콘용 정적 빌드를 제공하지 않습니다. Foundry는 [svm](https://github.com/alloy-rs/svm-rs)을 사용하여 Apple 실리콘용 네이티브 빌드를 설치합니다.

모든 solc 버전은 디렉토리가 이미 존재하는 경우 `~/.svm/` 아래에 설치됩니다. 그렇지 않으면 `$XDG_DATA_HOME/svm/`을 사용하며, 이는 리눅스에서는 보통 `$HOME/.local/share/svm/`에, MacOS에서는 `$HOME/Library/Application Support/svm/`에 매핑됩니다. `SolcError: ...`와 같은 solc 관련 오류가 발생하면 `~/.svm/` 디렉토리를 제거하고 다시 시도해 보세요. 이렇게 하면 새로운 설치가 트리거되어 일반적으로 문제가 해결됩니다.

Apple 실리콘을 사용하는 경우 [`z3` 정리 증명기(theorem prover)](https://github.com/Z3Prover/z3)가 설치되어 있는지 확인하세요: `brew install z3`

> **참고**: 네이티브 Apple 실리콘 빌드는 `0.8.5` 이상부터만 사용할 수 있습니다. 이전 버전이 필요한 경우 Apple 실리콘 rosetta를 활성화하여 실행해야 합니다.

### JavaScript 모노레포(`pnpm`)에서 Forge가 실패합니다

`pnpm`과 같은 매니저는 `node_modules` 폴더를 관리하기 위해 심볼릭 링크를 사용합니다.

일반적인 레이아웃은 다음과 같을 수 있습니다:

```text
├── contracts
│    ├── contracts
│    ├── foundry.toml
│    ├── lib
│    ├── node_modules
│    ├── package.json
├── node_modules
│    ├── ...
├── package.json
├── pnpm-lock.yaml
├── pnpm-workspace.yaml
```

여기서 Foundry 작업 공간은 `./contracts`에 있지만, `./contracts/node_modules`의 패키지는 `./node_modules`로 심볼릭 링크되어 있습니다.

`./contracts/node_modules`에서 [`forge build`][forge-build]를 실행할 때 다음과 같은 오류가 발생할 수 있습니다:

```console
error[6275]: ParserError: Source "node_modules/@openzeppelin/contracts/utils/cryptography/draft-EIP712.sol" not found: File outside of allowed directories. The following are allowed: "<repo>/contracts", "<repo>/contracts/contracts", "<repo>/contracts/lib".
 --> node_modules/@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol:8:1:
  |
8 | import "../../../utils/cryptography/draft-EIP712.sol";
```

이 오류는 `solc`가 심볼릭 링크된 파일을 확인할 수 있었지만 Foundry 작업 공간(`./contracts`) 외부에 있을 때 발생합니다.

`foundry.toml`의 `allow_paths`에 `node_modules`를 추가하면 solc에 해당 디렉토리에 대한 액세스 권한을 부여하여 읽을 수 있게 됩니다:

```toml
# 이것은 `solc --allow-paths ../node_modules`로 번역됩니다
allow_paths = ["../node_modules"]
```

경로는 Foundry 작업 공간에 상대적이라는 점에 유의하세요. [solc allowed-paths](https://docs.soliditylang.org/en/latest/path-resolution.html#allowed-paths)도 참조하세요.

### `Permission denied (os error 13)` 오류가 발생합니다

다음과 같은 오류가 표시되는 경우

```console
Failed to create artifact parent folder "/.../MyProject/out/IsolationModeMagic.sol": Permission denied (os error 13)
```

폴더 권한 문제일 가능성이 높습니다. `user`가 프로젝트 루트 폴더에 대한 쓰기 권한이 있는지 확인하세요.

리눅스에서 경로를 정규화하면 이상한 경로(`/_1/...`)가 생성될 수 있다는 [보고](https://github.com/foundry-rs/foundry/issues/3268)가 있습니다. 전체 프로젝트 폴더를 삭제하고 다시 초기화하면 해결될 수 있습니다.

### `forge build` 실행 시 연결 거부됨(Connection refused)

[`forge build`][forge-build]에 의해 호출된 github URL에 액세스할 수 없는 경우 다음과 같은 오류가 표시됩니다.

```console
Error:
error sending request for url (https://raw.githubusercontent.com/roynalnaruto/solc-builds/ff4ea8a7bbde4488428de69f2c40a7fc56184f5e/macosx/aarch64/list.json): error trying to connect: tcp connect error: Connection refused (os error 61)
```

사용자 위치에서의 URL 액세스가 제한되어 있어 연결에 실패했습니다. 이 문제를 해결하려면 프록시를 설정해야 합니다.

터미널에서 먼저 `export http_proxy=http://127.0.0.1:7890 https_proxy=http://127.0.0.1:7890`을 실행한 다음 [`forge build`][forge-build]를 성공적으로 실행할 수 있습니다.

### 테스트에서 `[NotActivated] EvmError: NotActivated` 오류가 발생합니다.

이 오류는 EVM 버전 불일치를 나타냅니다. `evm_version` 구성이 사용 중인 테스트(포크된 체인)와 일치하는지 확인하세요(`prevrandao not set`과 같은 오류의 경우도 마찬가지입니다). [`evm_version` 설정](/config/reference/solidity-compiler#evm_version)을 참조하세요.

[tg-support]: https://t.me/foundry_support
[forge-test]: /forge/tests/overview
[traces]: /forge/tests/traces
[config-solc]: /config/reference/solidity-compiler#solc_version
[config]: /config/overview
[forge-build]: /forge/reference/build
[console-log]: /reference/forge-std/console-log
[forge-std]: https://github.com/foundry-rs/forge-std
[dstestplus]: https://github.com/transmissions11/solmate/blob/19a4f345970ed39ee6369f343d145e0d4071c18a/src/test/utils/DSTestPlus.sol#L10
