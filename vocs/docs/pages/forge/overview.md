---
description: Forge는 퍼징 및 가스 추적과 같은 고급 기능을 사용하여 스마트 컨트랙트를 빌드, 테스트 및 배포하기 위한 커맨드 라인 도구입니다.
---

## Forge

Forge는 Foundry와 함께 제공되는 커맨드 라인 도구입니다. Forge는 스마트 컨트랙트를 테스트, 빌드 및 배포합니다.

Forge는 Foundry 제품군의 일부이며 `cast`, `chisel`, `anvil`과 함께 설치됩니다. 아직 Foundry를 설치하지 않았다면 [Foundry 설치](/introduction/installation)를 참조하세요.

### 시작하기

Forge를 이해하는 가장 좋은 방법은 (30초 이내에!) 직접 사용해 보는 것입니다.

먼저, 새로운 `counter` 예제 저장소를 초기화해 보겠습니다:

```sh
forge init counter
```

다음으로 `counter` 디렉토리로 이동(`cd`)하여 빌드합니다:

```sh
forge build
```

```console
[⠊] Compiling...
[⠔] Compiling 27 files with Solc 0.8.28
[⠒] Solc 0.8.28 finished in 452.13ms
Compiler run successful!
```

컨트랙트를 [테스트](https://book.getfoundry.sh/forge/tests#tests)해 보겠습니다:

```sh
forge test
```

```console
[⠊] Compiling...
No files changed, compilation skipped

Ran 2 tests for test/Counter.t.sol:CounterTest
[PASS] testFuzz_SetNumber(uint256) (runs: 256, μ: 31121, ~: 31277)
[PASS] test_Increment() (gas: 31293)
Suite result: ok. 2 passed; 0 failed; 0 skipped; finished in 5.35ms (4.86ms CPU time)

Ran 1 test suite in 5.91ms (5.35ms CPU time): 2 tests passed, 0 failed, 0 skipped (2 total tests)
```

마지막으로 배포 스크립트를 실행해 보겠습니다:

```sh
forge script script/Counter.s.sol
```

```
[⠊] Compiling...
No files changed, compilation skipped
Script ran successfully.
Gas used: 109037

If you wish to simulate on-chain transactions pass a RPC URL.
```

:::info
사용 가능한 모든 하위 명령어에 대한 전체 개요는 [`forge` 참조](/forge/reference/forge)를 확인하세요.
:::
