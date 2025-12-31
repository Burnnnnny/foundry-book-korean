## `foundry.toml` 구성하기

Forge는 프로젝트 루트에 위치한 `foundry.toml`이라는 구성 파일을 사용하여 설정할 수 있습니다.

구성은 프로필별로 네임스페이스를 지정할 수 있습니다. 기본 프로필의 이름은 `default`이며, 다른 모든 프로필은 이를 상속받습니다. `default` 프로필을 자유롭게 사용자 정의할 수 있으며, 필요한 만큼 새로운 프로필을 추가할 수 있습니다.

또한, 홈 디렉토리에 전역 `foundry.toml`을 생성할 수도 있습니다.

두 개의 프로필을 포함하는 구성 파일을 살펴보겠습니다. 하나는 항상 최적화 도구를 활성화하는 기본 프로필이고, 다른 하나는 항상 트레이스를 표시하는 CI 프로필입니다:

```toml
[profile.default]
optimizer = true
optimizer_runs = 20_000

[profile.ci]
verbosity = 4
```

`forge`를 실행할 때, `FOUNDRY_PROFILE` 환경 변수를 사용하여 사용할 프로필을 지정할 수 있습니다.

### 독립형 섹션

프로필 섹션 외에도 구성 파일에는 독립형 섹션(`[fmt]`, `[fuzz]`, `[invariant]` 등)이 포함될 수 있습니다. 기본적으로 각 독립형 섹션은 `default` 프로필에 속합니다.
즉, `[fmt]`는 `[profile.default.fmt]`와 동일합니다.

`default` 이외의 다른 프로필에 대해 독립형 섹션을 구성하려면 `[profile.<profile name>.<standalone>]` 구문을 사용하세요.
예: `[profile.ci.fuzz]`.

<br></br>

:::info
구성할 수 있는 모든 항목에 대한 전체 개요는 [`foundry.toml` 참조](/config/reference/default-config)를 확인하세요.
:::
