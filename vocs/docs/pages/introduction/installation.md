---
description: Foundryup, 미리 컴파일된 바이너리, 또는 소스에서 빌드하여 시스템에 Foundry를 설치하는 전체 가이드입니다.
---

## 설치

설치 중 문제가 발생하면 [FAQ](/misc/faq)를 참조하여 도움을 받으세요.

### 미리 컴파일된 바이너리

미리 컴파일된 바이너리는 [GitHub 릴리스 페이지](https://github.com/foundry-rs/foundry/releases)에서 다운로드할 수 있습니다. 쉬운 관리를 위해 [Foundryup 사용](#using-foundryup)을 권장합니다.

### Foundryup 사용하기

Foundryup은 Foundry 도구 모음의 공식 설치 프로그램입니다. 자세한 내용은 [여기](https://github.com/foundry-rs/foundry/blob/master/foundryup/README.md)에서 확인할 수 있습니다.

Foundryup을 설치하려면 터미널을 열고 다음 명령어를 실행하세요:

```sh
curl -L https://foundry.paradigm.xyz | bash
```

이 명령어는 Foundryup을 설치합니다. 화면의 지시를 따르기만 하면 CLI에서 `foundryup` 명령어를 사용할 수 있게 됩니다.

`foundryup`을 실행하면 [미리 컴파일된 바이너리](#precompiled-binaries)인 `forge`, `cast`, `anvil`, `chisel`의 최신 `stable` 버전이 자동으로 설치됩니다. 최신 `nightly` 빌드를 사용하려면 `foundryup --install nightly`를 실행하세요. 특정 버전이나 커밋을 설치하는 등의 추가 옵션은 `foundryup --help`를 실행하여 확인하세요.

[Tempo의 Foundry 포크](https://github.com/tempoxyz/tempo-foundry)에서 배포하는 바이너리를 설치하려면 다음을 실행하세요:

```sh
foundryup -n tempo
```

이 명령어는 [미리 컴파일된 바이너리](#precompiled-binaries)인 `forge`와 `cast`의 최신 `nightly` 버전을 설치합니다.

> ℹ️ **참고**
> Windows 사용자의 경우, Foundryup이 현재 Powershell이나 명령 프롬프트(Cmd)를 지원하지 않으므로 터미널로 [Git BASH](https://gitforwindows.org/) 또는 [WSL](https://learn.microsoft.com/en-us/windows/wsl/install)을 설치하여 사용해야 합니다.

#### 바이너리의 무결성 및 출처 검증

Foundry 바이너리는 [GitHub 아티팩트 증명](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations/using-artifact-attestations-to-establish-provenance-for-builds)을 사용하여 증명됩니다.

`foundryup`을 사용하여 릴리스를 설치할 때, 바이너리의 해시를 Github 증명 아티팩트의 해시와 대조합니다. 이는 바이너리가 공식 Foundry 저장소에서 자동화된 릴리스 프로세스에 의해 빌드되었음을 보장합니다. 또한 이 해시를 사용하여 해당 버전의 바이너리가 이미 설치되어 있는지 확인합니다. 만약 설치되어 있다면 다운로드를 건너뜁니다.

`--force` 플래그를 전달하여 검증 단계를 건너뛸 수 있습니다. 또한 대조할 해시가 없기 때문에 설치 프로그램이 바이너리의 새 버전을 강제로 설치하게 됩니다.

현재 `foundryup`이 검증할 `*attestation.txt`가 없는 이전 릴리스를 설치하는 경우에는 설치 시 검증이 건너뛰어질 수 있습니다. 다음과 같이 언제든지 무결성을 수동으로 검증할 수 있습니다:

예를 들어 `forge`의 경우:

```shell
gh attestation verify --owner foundry-rs $(which forge)

✓ Verification succeeded!

The following 1 attestation matched the policy criteria

- Attestation #1
  - Build repo:..... foundry-rs/foundry
  - Build workflow:. .github/workflows/release.yml@refs/tags/stable
  - Signer repo:.... foundry-rs/foundry
  - Signer workflow: .github/workflows/release.yml@refs/tags/stable
```

이는 이전 릴리스의 경우에도 마찬가지입니다.

### 소스에서 빌드하기

#### 필수 조건

[Rust](https://rust-lang.org) 컴파일러와 Rust의 패키지 관리자인 Cargo가 필요합니다. 가장 쉬운 설치 방법은 [`rustup.rs`](https://rustup.rs/)를 사용하는 것입니다.

Foundry는 일반적으로 최신 안정 버전의 Rust로 빌드하는 것만 지원합니다. 이전 버전의 Rust를 사용 중이라면 `rustup`으로 업데이트할 수 있습니다:

```sh
rustup update stable
```

Windows 사용자의 경우, "C++를 사용한 데스크톱 개발" 워크로드가 설치된 최신 버전의 [Visual Studio](https://visualstudio.microsoft.com/downloads/)도 필요합니다.

#### 빌드하기

[Foundryup](#using-foundryup)에서 제공하는 다양한 플래그를 사용할 수 있습니다:

```sh
foundryup --branch master
foundryup --path path/to/foundry
```

또는 다음 명령어로 Cargo를 통해 설치할 수 있습니다:

```sh
cargo install --git https://github.com/foundry-rs/foundry --profile release --locked forge cast chisel anvil
```

[Foundry 저장소](https://github.com/foundry-rs/foundry)의 로컬 복사본에서 수동으로 빌드할 수도 있습니다:

```sh
# 저장소 복제
git clone https://github.com/foundry-rs/foundry.git
cd foundry
# Forge 설치
cargo install --path ./crates/forge --profile release --force --locked
# Cast 설치
cargo install --path ./crates/cast --profile release --force --locked
# Anvil 설치
cargo install --path ./crates/anvil --profile release --force --locked
# Chisel 설치
cargo install --path ./crates/chisel --profile release --force --locked
```

### GitHub Actions를 사용한 CI 설치

CI 파이프라인에서 Foundry를 설정하는 방법은 [foundry-rs/foundry-toolchain](https://github.com/foundry-rs/foundry-toolchain) GitHub Action을 참조하세요.

### Docker로 Foundry 사용하기

:::note
M1 칩을 포함한 일부 시스템에서는 Docker 이미지를 로컬에서 빌드할 때 문제가 발생할 수 있습니다. 이는 알려진 문제입니다.
:::

Foundry는 Docker 컨테이너 내부에서도 실행할 수 있습니다. Docker가 설치되어 있지 않다면 [Docker 웹사이트](https://docs.docker.com/get-docker/)에서 다운로드할 수 있습니다.

Docker가 설치되면 다음을 실행하여 최신 Foundry 릴리스를 가져올 수 있습니다:

```sh
docker pull ghcr.io/foundry-rs/foundry:latest
```

Foundry 저장소에서 다음 명령어를 실행하여 로컬에서 Docker 이미지를 빌드할 수도 있습니다:

```sh
docker build -t foundry .
```

이 이미지 사용에 대한 예제와 가이드는 [Docker 가이드](/guides/foundry-in-docker)를 참조하세요.

### 제거하기

Foundry는 모든 것을 `.foundry` 디렉토리에 포함하며, 이는 일반적으로 Linux의 경우 `/home/<user>/.foundry/`, MacOS의 경우 `/Users/<user>/.foundry/`, Windows의 경우 `C:\Users\<user>\.foundry`에 위치합니다. 여기서 `<user>`는 사용자 이름입니다.

Foundry를 제거하려면 `.foundry` 디렉토리를 삭제하세요.

:::warning
.foundry 디렉토리에는 키스토어가 포함될 수 있습니다. 유지하고 싶은 키스토어는 반드시 백업하세요.
:::

PATH에서 Foundry 제거:

- 선택적으로 셸 구성 파일(`.bashrc`, `.zshrc` 등)에서 Foundry를 제거할 수 있습니다. 이를 위해 Foundry를 PATH에 추가하는 줄을 삭제하세요:

```sh
export PATH="$PATH:/home/user/.foundry/bin"
```
