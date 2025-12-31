## VSCode와 통합하기

[VSCode Solidity 확장 프로그램](https://github.com/juanfranblanco/vscode-solidity)을 설치하여 Visual Studio Code에서 Solidity 지원을 받을 수 있습니다.

확장 프로그램이 Foundry와 잘 작동하게 하려면 몇 가지 조정이 필요할 수 있습니다.

### 1. 리매핑 (Remappings)

리매핑을 `remappings.txt`에 배치하는 것이 좋습니다.

이미 `foundry.toml`에 있다면, 복사하여 `remappings.txt`를 대신 사용하세요. Foundry가 제공하는 자동 생성 리매핑을 사용하려면 `forge remappings > remappings.txt`를 실행하세요.

### 2. 의존성 (Dependencies)

확장 프로그램이 의존성을 찾을 수 있도록 `.vscode/settings.json`에 다음을 추가해야 할 수 있습니다:

```json
{
  "solidity.packageDefaultDependenciesContractsDirectory": "src",
  "solidity.packageDefaultDependenciesDirectory": "lib"
}
```

여기서 `src`는 소스 코드 디렉토리이고 `lib`은 의존성 디렉토리입니다.

### 3. 포매터 (Formatter)

저장 시 Foundry에 내장된 포매터가 코드를 자동으로 포맷하도록 하려면, `.vscode/settings.json`에 다음 설정을 추가할 수 있습니다:

```json
{
  "editor.formatOnSave": true,
  "[solidity]": {
    "editor.defaultFormatter": "JuanBlanco.solidity"
  },
  "solidity.formatter": "forge"
}
```

포매터 설정을 구성하려면, [포매터](/config/reference/formatter) 참조를 확인하세요.

### 4. Solc 버전

마지막으로, Solidity 컴파일러 버전을 지정하는 것이 좋습니다:

```json
"solidity.compileUsingRemoteVersion": "v0.8.17"
```

Foundry를 선택한 버전에 맞추려면, `foundry.toml`의 `default` 프로필에 다음을 추가하세요.

```toml
solc = "0.8.17"
```

### OpenZeppelin 컨트랙트 및 비표준 프로젝트 레이아웃 사용 예시

```bash
.
└── project
    └── contracts
        ├── lib
        │   ├── forge-std
        │   └── openzeppelin-contracts
        ├── script
        ├── src
        └── test
```

`remappings.txt` 파일에 줄 추가 ([`forge remapping`](/guides/project-setup/dependencies#remapping-dependencies)):

```solidity
@openzeppelin/=lib/openzeppelin-contracts/
```

`.vscode/settings.json` 파일에 줄 추가 (solidity 확장 설정):

```json
{
  "solidity.remappings": [
    "@openzeppelin/=project/contracts/lib/openzeppelin-contracts/"
  ]
}
```

이제 OpenZeppelin 문서의 모든 컨트랙트를 사용할 수 있습니다.

```javascript
import { ERC20 } from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```
