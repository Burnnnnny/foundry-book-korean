---
description: 컨트랙트, 테스트, 종속성 및 구성 파일을 위한 디렉토리로 Foundry 프로젝트 구조를 구성합니다.
---

## 프로젝트 레이아웃

Forge는 프로젝트를 구조화하는 방식에 있어 유연합니다. 기본 구조는 다음과 같습니다:

```sh
// [!include ~/snippets/output/hello_foundry/tree-with-files:output]
```

- `foundry.toml`을 사용하여 Foundry의 동작을 구성할 수 있습니다. `extends` 필드를 사용하여 기본 파일에서 구성을 확장할 수 있습니다 (자세한 내용은 아래 [구성 상속](#configuration-inheritance) 참조).
- 리매핑은 `remappings.txt`에 지정됩니다.
- 컨트랙트의 기본 디렉토리는 `src/`입니다.
- 테스트의 기본 디렉토리는 `test/`이며, `test`로 시작하는 함수가 있는 모든 컨트랙트는 테스트로 간주됩니다.
- 종속성은 `lib/`에 git 서브모듈로 저장됩니다.

`--lib-paths` 및 `--contracts` 플래그를 사용하여 Forge가 종속성과 컨트랙트를 찾는 위치를 각각 구성할 수 있습니다. 또는 `foundry.toml`에서 구성할 수도 있습니다.

리매핑과 결합하면 Hardhat 및 Truffle과 같은 다른 툴체인의 프로젝트 구조를 지원하는 데 필요한 유연성을 얻을 수 있습니다.

자동 Hardhat 지원을 위해 `--hh` 플래그를 전달할 수도 있으며, 이는 `--lib-paths node_modules --contracts contracts` 플래그를 설정합니다.

## 구성 상속 (Configuration Inheritance)

Foundry는 `foundry.toml`의 `extends` 필드를 통해 구성 상속을 지원합니다. 이를 통해 여러 프로젝트나 프로필에서 공유할 수 있는 기본 구성 파일을 유지할 수 있습니다.

### 기본 사용법

`foundry.toml`에서 간단한 문자열이나 객체 형식을 사용하여 상속할 기본 구성 파일을 지정할 수 있습니다:

```toml
# 간단한 문자열 형식 (기본 "extend-arrays" 전략 사용)
[profile.default]
extends = "./base-config.toml"
src = "src"
test = "test"
```

또는 명시적인 병합 전략 사용:

```toml
# 전략이 포함된 객체 형식
[profile.default]
extends = { path = "./base-config.toml", strategy = "replace-arrays" }
src = "src"
test = "test"
```

기본 구성 파일(`base-config.toml`)에는 유효한 Foundry 구성이 포함될 수 있습니다:

```toml
[profile.default]
solc = "0.8.23"
optimizer = true
optimizer_runs = 200
```

### 우선순위 규칙

구성 값은 다음 우선순위(높은 순에서 낮은 순)로 해결됩니다:
1. **환경 변수** - 항상 가장 높은 우선순위를 가짐
2. **로컬 `foundry.toml`** - 프로젝트의 메인 구성 파일에 있는 값
3. **기본/상속된 파일** - `extends`에 지정된 파일의 값

### 병합 전략

객체 형식에서 전략을 지정하여 구성이 병합되는 방식을 제어할 수 있습니다:

```toml
[profile.default]
extends = { path = "./base-config.toml", strategy = "extend-arrays" }
```

사용 가능한 전략:
- **`extend-arrays`** (기본값): 배열이 연결되고(기본 요소 + 로컬 요소), 다른 값은 교체됩니다.
- **`replace-arrays`**: 배열이 로컬 배열로 완전히 교체되고, 다른 값도 교체됩니다.
- **`no-collision`**: 기본 파일과 로컬 파일 모두에 키가 존재하면 오류를 발생시킵니다.

### 프로필별 상속

각 프로필은 서로 다른 구성 파일에서 상속받을 수 있습니다:

```toml
[profile.default]
extends = "./base-config.toml"

[profile.production]
extends = "./production-base.toml"
```

### 중요 참고 사항

- `extends` 경로는 `foundry.toml` 파일의 위치를 기준으로 상대적이어야 합니다.
- 상속된 파일은 어떤 이름이든 가질 수 있습니다 (`foundry.toml`로 제한되지 않음).
- 상속된 파일 자체는 다른 파일에서 상속받을 수 없습니다 (중첩 상속은 허용되지 않음).
- 상속은 `foundry.toml`에 선택된 프로필이 존재하고 `extends` 필드가 구성된 경우에만 적용됩니다.
