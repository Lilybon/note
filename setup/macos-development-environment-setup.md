# MacOS Development Environment Setup

Just a memo for work.

### Package management

1. Download [**brew**](https://brew.sh/index_zh-tw).

### Terminal

1. Download **iTerm2** through brew.
2. Download [**iTerm2 color scheme**](https://iterm2colorschemes.com/) and apply on iTerm2.
3. Download [**powerlevel10k**](https://github.com/romkatv/powerlevel10k) and its recommendated font.
4. Download plugins.
    * [autojump](https://github.com/wting/autojump)
    * [zsh-autosuggestion](https://github.com/zsh-users/zsh-autosuggestions)
    * zsh-syntax-highlighting
5. Edit .zshrc configuration, here is an example below:
    ```
    ZSH_THEME="powerlevel10k/powerlevel10k"
    POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(context dir dir_writable vcs vi_mode)
    POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status background_jobs history ram load time)

    plugins=(git autojump brew zsh-autosuggestions zsh-syntax-highlighting)
    ```

### Git

1. Download [**git**](https://git-scm.com/).
2. Setup user name and email.
    ```bash
    git config --global user.name "USER_NAME"
    git config --global user.email "USER_EMAIL"
    ```
3. Setup [**SSH Key**](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).
4. Download [**Fork**](https://git-fork.com/).

### VSCode

1. Download [**VSCode**](https://code.visualstudio.com/).
2. Download plugins.
    * Auto Close Tag
    * Auto Rename Tag
    * DotENV
    * EditorConfig
    * ESLint
    * GitLens
    * One Dark Pro
    * Vetur (for Vue2 dev)
    * Volar (for Vue3 dev)

### XCode

1. Confirm that macOS on your device is latest.
2. Download **XCode** through appstore.
3. Download **XCode select**.
    ```bash
    xcode-select --install
    ```
4. [Launch iOS simulator](https://stackoverflow.com/questions/31179706/how-can-i-launch-the-ios-simulator-from-terminal).

### Node

1. Download [**nvm**](https://github.com/nvm-sh/nvm).
2. Download **LTS node** through nvm.
    ```bash
    git config --global user.name "USER_NAME"
    git config --global user.email "USER_EMAIL"
    ```
3. Download **yarn** through npm
    ```bash
    npm install --global yarn
    ```

### Flutter

1. [MacOS setup](https://hackmd.io/pxaa10MtTQ2eCtqylZAzEg)

###### tags: `Setup`

