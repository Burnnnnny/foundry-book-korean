## 동적 테스트 링킹 (Dynamic Test Linking)

[v1.1 릴리스](https://github.com/foundry-rs/foundry/releases/tag/v1.1.0)에는 [Solar](https://github.com/paradigmxyz/solar)를 기반으로 구축된 동적 테스트 링킹 기능이 포함되어 있습니다. 이 기능은 컨트랙트 로직을 변경할 때 중복 테스트 컴파일을 제거하여 Foundry가 대규모 테스트 스위트의 재컴파일을 건너뛰게 함으로써 컴파일 시간을 획기적으로 단축합니다.

작동 방식:

초기 빌드 시, Foundry는 테스트 중인 컨트랙트의 생성자 매개변수를 추출하고 직접 인스턴스화를 [`deployCode` 칝코드](/reference/cheatcodes/get-deployed-code)로 대체하여 테스트 컨트랙트를 전처리합니다.

이후 컴파일에서는 소스와 모든 관련 테스트 컨트랙트를 다시 컴파일하는 대신 배포된 컨트랙트에 대해 미리 빌드된 아티팩트를 재사용합니다.
동적 테스트 링킹 기능은 매우 빠르고 모듈화된 Solidity 컴파일러인 [Solar](https://github.com/paradigmxyz/solar)를 기반으로 구축되었습니다.

`foundry.toml` 파일에서 `dynamic_test_linking` 구성 옵션을 `true`로 설정하여 이 기능을 활성화할 수 있습니다:

```toml
[profile.default]
...
dynamic_test_linking = true
```

또는 `forge build` 명령어에 `--dynamic-test-linking` 플래그를 전달하여 활성화할 수 있습니다:

```bash
forge build --dynamic-test-linking
```

향후 이 기능을 기본값으로 활성화하는 것을 검토 중입니다.

[PR](https://github.com/foundry-rs/foundry/pull/10010)의 벤치마크에 따르면 대규모 프로젝트에서 컴파일 시간이 10배 이상 단축되는 것으로 나타났습니다:

| 프로젝트                                                                                                                  | 변경 사항                                                                                                                                                                              | 컴파일된 파일 (사용/미사용, 초기 컴파일 후) | 컴파일 시간 (사용/미사용, 초기 컴파일 후) |
| ------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------- |
| [uniswap v4-core](https://github.com/Uniswap/v4-core/tree/80311e34080fee64b6fc6c916e9a51a437d0e482)                      | [PoolManager.sol#L107](https://github.com/Uniswap/v4-core/blob/80311e34080fee64b6fc6c916e9a51a437d0e482/src/PoolManager.sol#L107)에 `Lock.lock();` 추가                             | 1 / 19                                                 | 2.25s / 165.13s                                         |
| [spark-psm](https://github.com/sparkdotfi/spark-psm/tree/9d0bcc045e81407408368c9a4bb6e3f13db77e32)                       | [PSM3.sol#L125](https://github.com/sparkdotfi/spark-psm/blob/9d0bcc045e81407408368c9a4bb6e3f13db77e32/src/PSM3.sol#L125)에서 `amountOut < minAmountOut` 변경                       | 3 / 28                                                 | 2.14s / 16.15s                                          |
| [morpho-blue-bundlers](https://github.com/morpho-org/morpho-blue-bundlers/tree/1fa17256abb86c4de48fd5e251ebd46aae70ca1a) | [MorphoBundler.sol#L106](https://github.com/morpho-org/morpho-blue-bundlers/blob/1fa17256abb86c4de48fd5e251ebd46aae70ca1a/src/MorphoBundler.sol#L106)에서 `if (assets < 0)` 변경   | 11 / 36                                                | 16.39s / 251.05s                                        |
| [morpho-blue](https://github.com/morpho-org/morpho-blue/commit/9e2b0755b47bbe5b09bf1be8f00e060d4eab6f1c)                 | [Morpho.sol#L424](https://github.com/morpho-org/morpho-blue/blob/9e2b0755b47bbe5b09bf1be8f00e060d4eab6f1c/src/Morpho.sol#L424)에 `require(assets != 0, ErrorsLib.ZERO_ASSETS)` 추가 | 1 / 23                                                 | 1.01s / 133.73s                                         |
| [sablier lockup](https://github.com/sablier-labs/lockup/tree/b2f33926fcac72a1a855c6b8ccaa75166895f13c)                   | [SablierLockup.sol#L480](https://github.com/sablier-labs/lockup/blob/b2f33926fcac72a1a855c6b8ccaa75166895f13c/src/SablierLockup.sol#L480)에서 `if (cliffTime < 0)` 변경            | 1 / 104                                                | 781ms / 71.29s                                          |
| [solady](https://github.com/Vectorized/solady/commit/724c39bdfebb593157c2dfa6797c07a25dfb564c)                           | [Ownable.sol#L182](https://github.com/Vectorized/solady/blob/724c39bdfebb593157c2dfa6797c07a25dfb564c/src/auth/Ownable.sol#L182)에 추가적인 `_setOwner(newOwner)` 추가            | 9 / 14                                                 | 6.17s / 6.34s                                           |
| [euler evc](https://github.com/euler-xyz/ethereum-vault-connector/commit/64f6d2171a57e02a0f95bcbdecf1d92e9d253d40)       | [Set.sol#L7](https://github.com/euler-xyz/ethereum-vault-connector/blob/64f6d2171a57e02a0f95bcbdecf1d92e9d253d40/src/Set.sol#L7)에서 `SET_MAX_ELEMENTS`를 `11`로 변경               | 28 / 30                                                | 9.17s / 9.40s                                           |
