## 셸 자동 완성 (Shell Autocompletion)

`bash`, `elvish`, `fish`, `powershell`, `zsh`에 대한 자동 완성 셸 스크립트를 생성할(generate) 수 있습니다.

### zsh

먼저, `~/.zshrc` 파일에 다음 내용이 있는지 확인하세요 (없다면 추가하세요):

```sh
autoload -U compinit
compinit -i
```

그 다음 실행하세요:

```sh
forge completions zsh | sudo tee /usr/local/share/zsh/site-functions/_forge
cast completions zsh | sudo tee /usr/local/share/zsh/site-functions/_cast
anvil completions zsh | sudo tee /usr/local/share/zsh/site-functions/_anvil
```

macOS의 경우:

```sh
forge completions zsh > /opt/homebrew/completions/zsh/_forge
cast completions zsh > /opt/homebrew/completions/zsh/_cast
anvil completions zsh > /opt/homebrew/completions/zsh/_anvil
```

### fish

```sh
mkdir -p $HOME/.config/fish/completions
forge completions fish > $HOME/.config/fish/completions/forge.fish
cast completions fish > $HOME/.config/fish/completions/cast.fish
anvil completions fish > $HOME/.config/fish/completions/anvil.fish
source $HOME/.config/fish/config.fish
```

### bash

```sh
mkdir -p $HOME/.local/share/bash-completion/completions
forge completions bash > $HOME/.local/share/bash-completion/completions/forge
cast completions bash > $HOME/.local/share/bash-completion/completions/cast
anvil completions bash > $HOME/.local/share/bash-completion/completions/anvil
exec bash
```
