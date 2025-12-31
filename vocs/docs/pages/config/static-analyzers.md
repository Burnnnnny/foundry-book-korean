## 정적 분석 도구

### Slither

[slither](https://github.com/crytic/slither)를 사용하여 프로젝트를 테스트하려면, 다음은 `slither.config.json`의 예시입니다:

```json
{
  "filter_paths": "lib"
}
```

전체 프로젝트에 대해 Slither를 실행하려면, 프로젝트 루트에서 다음 명령어를 사용하세요:

```sh
slither .
```

기본적으로(버전 0.10.0 기준), 테스트와 스크립트는 건너뜁니다. 테스트와 스크립트를 포함하려면 `--foundry-compile-all` 플래그를 추가하세요.

단일 파일에 대해 Slither를 실행하려면 다음 명령어를 사용하세요:

```sh
slither src/Contract.sol
```

참고로, 이를 위해서는 [foundry 설정 파일에서 solc 버전](https://book.getfoundry.sh/config/reference/solidity-compiler#solc_version)을 구성해야 합니다.

`solc_remaps` 옵션을 통해 리매핑을 제공할 필요는 없습니다. Slither가 Foundry 프로젝트에서 자동으로 리매핑을 감지합니다. Slither는 빌드를 수행하기 위해 `forge`를 호출합니다.

더 많은 정보는 [Slither 위키](https://github.com/crytic/slither/wiki/Usage)를 참조하세요.

위에서 언급한 `slither.config.json` 예시와 같은 사용자 정의 설정을 사용하려면, [slither-wiki](https://github.com/crytic/slither/wiki/Usage#configuration-file)에 언급된 대로 다음 명령어를 사용합니다. 기본적으로 slither는 `slither.config.json`을 찾지만, 경로와 다른 `json` 파일을 지정할 수 있습니다:

```sh
slither --config-file <path>/file.config.json .
```

예시 출력 (Raw):

```bash
Pragma version^0.8.13 (Counter.sol#2) necessitates a version too recent to be trusted. Consider deploying with 0.6.12/0.7.6/0.8.7
solc-0.8.13 is not recommended for deployment
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-versions-of-solidity

setNumber(uint256) should be declared external:
        - Counter.setNumber(uint256) (Counter.sol#7-9)
increment() should be declared external:
        - Counter.increment() (Counter.sol#11-13)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#public-function-that-could-be-declared-external
Counter.sol analyzed (1 contracts with 78 detectors), 4 result(s) found
```

Slither는 CI/CD를 위한 [GitHub Action](https://github.com/marketplace/actions/slither-action)도 제공합니다.

### Aderyn

[aderyn](https://github.com/cyfrin/aderyn)을 사용하여 프로젝트를 테스트하려면, Cyfrin에서 지원하는 [VS Code 확장 프로그램](https://marketplace.visualstudio.com/items?itemName=Cyfrin.aderyn&ssr=false#overview)을 설치하세요.

도구를 수동으로 실행하려면, 비디오 가이드가 포함된 [빠른 시작](https://cyfrin.gitbook.io/cyfrin-docs/aderyn-cli/quickstart) 예시를 따르세요.

```bash
cd path/to/solidity/project/root
aderyn
```

더 많은 CLI 옵션은 [여기](https://cyfrin.gitbook.io/cyfrin-docs/cli-options)에서 확인하세요.

### Mythril

[mythril](https://github.com/ConsenSys/mythril)을 사용하여 프로젝트를 테스트하려면, 다음은 `mythril.config.json`의 예시입니다:

```json
{
  "remappings": ["ds-test/=lib/ds-test/src/", "forge-std/=lib/forge-std/src/"],
  "optimizer": {
    "enabled": true,
    "runs": 200
  }
}
```

참고로, `mythril`을 설치하려면 `rustc`를 nightly 버전으로 전환해야 합니다:

```
rustup default nightly
pip3 install mythril
myth analyze src/Contract.sol --solc-json mythril.config.json
```

더 많은 정보는 [mythril 문서](https://mythril-classic.readthedocs.io/en/develop/)를 참조하세요.

`--solc-json` 플래그를 사용하여 사용자 정의 Solc 컴파일러 출력을 Mythril에 전달할 수 있습니다. 예를 들어:

```bash
myth analyze src/Counter.sol --solc-json mythril.config.json
.
.
mythril.laser.plugin.loader [INFO]: Loading laser plugin: coverage
mythril.laser.plugin.loader [INFO]: Loading laser plugin: mutation-pruner
.
.
Achieved 11.56% coverage for code: 608060405234801561001057600080fd5b5060f78061001f6000396000f3fe6080604052348015600f57600080fd5b5060043610603c5760003560e01c80633fb5c1cb1460415780638381f58a146053578063d09de08a14606d575b600080fd5b6051604c3660046083565b600055565b005b605b60005481565b60405190815260200160405180910390f35b6051600080549080607c83609b565b9190505550565b600060208284031215609457600080fd5b5035919050565b60006001820160ba57634e487b7160e01b600052601160045260246000fd5b506001019056fea2646970667358221220659fce8aadca285da9206b61f95de294d3958c409cc3011ded856f421885867464736f6c63430008100033
mythril.laser.plugin.plugins.coverage.coverage_plugin [INFO]: Achieved 90.13% coverage for code: 6080604052348015600f57600080fd5b5060043610603c5760003560e01c80633fb5c1cb1460415780638381f58a146053578063d09de08a14606d575b600080fd5b6051604c3660046083565b600055565b005b605b60005481565b60405190815260200160405180910390f35b6051600080549080607c83609b565b9190505550565b600060208284031215609457600080fd5b5035919050565b60006001820160ba57634e487b7160e01b600052601160045260246000fd5b506001019056fea2646970667358221220659fce8aadca285da9206b61f95de294d3958c409cc3011ded856f421885867464736f6c63430008100033
mythril.laser.plugin.plugins.instruction_profiler [INFO]: Total: 1.0892839431762695 s
[ADD         ]   0.9974 %,  nr      9,  total   0.0109 s,  avg   0.0012 s,  min   0.0011 s,  max   0.0013 s
.
.
[SWAP1       ]   1.8446 %,  nr     18,  total   0.0201 s,  avg   0.0011 s,  min   0.0010 s,  max   0.0013 s
[SWAP2       ]   0.8858 %,  nr      9,  total   0.0096 s,  avg   0.0011 s,  min   0.0010 s,  max   0.0011 s

mythril.analysis.security [INFO]: Starting analysis
mythril.mythril.mythril_analyzer [INFO]: Solver statistics:
Query count: 61
Solver time: 3.6820807456970215
The analysis was completed successfully. No issues were detected.
```

결과가 있다면 출력 끝에 나열됩니다. 기본 `Counter.sol`에는 로직이 없으므로, `mythx`는 문제가 발견되지 않았다고 보고합니다.
