# Initial setup for new Macs running OS X 10.14+ for Python development and more

For fresh install of OS X; for personal use.


## General Mac stuff


#### Getting `Terminal.app` up and running

```bash
# show hidden files
defaults write com.apple.finder AppleShowAllFiles YES
# disable google chrome dark mode when Mojave dark mode is enabled
defaults write com.google.Chrome NSRequiresAquaSystemAppearance -bool yes
```


#### Other stuff

- `System Preferences/General/`
  - `Show scroll bars:` Always
  - `Click in the scroll bar to:` Jump to the spot that's clicked
  - `Recent items:` 50
- `System Preferences/Keyboard/`
  - Under `Text`, untick/remove everything
  - Under `Sources`, tick `All controls` on the bottom
  - Under `Input Sources`, set keyboard layout to U.S. (remove U.S. International)
- `System Preferences/Security & Privacy/`
  - Under `FileVault`, turn on FileVault
- `System Preferences/Accessibility/`
  - Under `Zoom`, tick `Use scroll gesture with modifier keys to zoom:`
  - Under `Display`, untick `Shake mouse pointer to locate`
  - Under `Mouse & Trackpad/Trackpad Options...`, tick `Enable dragging/three finger drag`
- `System Preferences/Software Update/`
  - Under `Advanced...`, untick `Download new updates when available`
- Finder preferences
  - `General`
    - `New Finder windows show:` home
  - `Advanced`
    - Tick `Show all filename extensions`
    - `When performing a search:` Search the Current Folder
- Finder View Options (Go home ⌘-⇧-H, then ⌘-J)
  - Tick `Always open in List View`
    - Tick `Browse in List View`
  - `Group by:` None
  - `Sort by:` Name
  - Tick `Calculate all sizes`
  - Tick `Show Library Folder`
  - `Use as Defaults`
- Finder `View` menu item
  - `Show Tab Bar`
  - `Show Path Bar`
  - `Show Status Bar`

- TextEdit preferences
  - `New Document`
    - `Format`: Plain text
    - Untick `Check spelling as you type`
  - `Open and Save`
    - Untick `Add ".txt" extension to plain text files`
    - Under `Plain Text File Encoding`, select two times `UTF-8`


## Terminal stuff


### Commandline-tools (including [SDK headers](https://github.com/pyenv/pyenv/wiki#suggested-build-environment)), Homebrew and it's essentials

```bash
# install homebrew and command-line tools, SDK headers
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
sudo installer -pkg /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg -target /
# check wether all is good
brew doctor
brew cask
brew tap buo/cask-upgrade  # `brew cu -a docker` - https://github.com/buo/homebrew-cask-upgrade#usage

# and some essentials
# - ruby, gcc-8 are linked in `.bash_profile`
# - node installs npm
brew install \
  git git-lfs bash-completion rsync curl openssl readline automake xz zlib \
  sshfs htop ncdu direnv pwgen \
  pyenv pyenv-virtualenv gcc@8 rust \
  ruby node yarn sqlite3

brew cask install iterm2
```

iTerm [nerd font](https://github.com/ryanoasis/nerd-fonts/blob/master/readme.md)
```sh
brew tap homebrew/cask-fonts
brew cask install font-inconsolatalgc-nerd-font
cargo install exa ripgrep  # ls, rg
```


#### Preferences

```bash
mkdir ~/git
git clone https://github.com/ddelange/new-mac-setup.git ~/git/new-mac-setup
ln -s ~/git/new-mac-setup/.bash_profile ~/.bash_profile
direnv edit ~  # add `export SECRET=42` to load global env vars

# Sublime Text 3 backup
# restore
cp -r "${HOME}/git/new-mac-setup/sublime_text_user_settings" "${HOME}/Library/Application Support/Sublime Text 3/Packages/User"
# create
cp -r "${HOME}/Library/Application Support/Sublime Text 3/Packages/User/" "${HOME}/git/new-mac-setup/sublime_text_user_settings/"

# Chrome search engines backup
# restore
sqlite3 "${HOME}/Library/Application Support/Google/Chrome/Default/Web Data" < ./search-engine-export.sql
# create
(printf 'begin transaction;\n'; sqlite3 "${HOME}/Library/Application Support/Google/Chrome/Default/Web Data" 'select short_name,keyword,url,favicon_url from keywords' | awk -F\| '{ printf "REPLACE INTO keywords (short_name, keyword, url, favicon_url) values ('"'"%s"'"', '"'"%s"'"', '"'"%s"'"', '"'"%s"'"');\n", $1, $2, $3, $4 }'; printf 'end transaction;\n') > ./search-engine-export.sql

git clone https://github.com/ddelange/yt.git ~/git/yt  && brew install youtube-dl
```
- iTerm2 preferences: under `General/Preferences`, tick `Load preferences from a custom folder or URL` and select `com.googlecode.iterm2.plist`
- iStat Menus preferences: `File/Import Settings...`, select `iStat Menus Settings.ismp`


### Git with 2FA

[Enable and set up 2FA](https://gist.github.com/ateucher/4634038875263d10fb4817e5ad3d332f). It's recommended to first delete any git configurations locally after enabling 2FA.

Use built-in keychain and app password from above, and add a Mac specific global gitignore:
```bash
git config --global user.name "ddelange"
git config --global user.email "14880945+ddelange@users.noreply.github.com"
git config --global credential.helper osxkeychain

curl -sLw "\n" "http://gitignore.io/api/macos,python,django,sublimetext" >> ~/.gitignore  # for all possibilities see http://gitignore.io/api/list
git config --global core.excludesfile "~/.gitignore"
```

Note: it's advised to add [commit signature verification](https://help.github.com/en/articles/managing-commit-signature-verification) to Git.
[Generate a GPG key](https://help.github.com/en/articles/generating-a-new-gpg-key#generating-a-gpg-key) and tell Git to use it:
```bash
brew install gpg
gpg --full-generate-key  # recommended settings: enter, 4096, enter
gpg --list-secret-keys --keyid-format LONG  # copy the key after 'sec  4096R/'
gpg --armor --export <key-here>  # paste this key at github.com/settings/keys
git config --global user.signingkey <key-here>
git config --global commit.gpgsign true
# sign tags using git tag -s
```

To [enable password caching](https://stackoverflow.com/a/38422272/5511061) for 1 week:
```bash
echo "default-cache-ttl 604800" >> ~/.gnupg/gpg-agent.conf
echo "max-cache-ttl 604800" >> ~/.gnupg/gpg-agent.conf
echo "log-file /var/log/gpg-agent.log" >> ~/.gnupg/gpg-agent.conf
```


##### Mac OSX specifics

```bash
git config --global core.trustctime false  # http://www.git-tower.com/blog/make-git-rebase-safe-on-osx
git config --global core.precomposeunicode false  # http://michael-kuehnel.de/git/2014/11/21/git-mac-osx-and-german-umlaute.html
git config --global core.untrackedCache true  # https://git-scm.com/docs/git-update-index#_untracked_cache
git config --global merge.log true  # Include summaries of merged commits in newly created merge commit messages
git config --global push.default "simple" # https://git-scm.com/docs/git-config#Documentation/git-config.txt-pushdefault
git config --global push.followTags true # https://git-scm.com/docs/git-config#Documentation/git-config.txt-pushfollowTags
```


##### Split diff

```bash
pip install git+https://github.com/jeffkaufman/icdiff.git  # usage: 'git df' using 'less' or 'git icdiff' without 'less'
git config --global --replace-all core.pager 'less -+$LESS -eFRSX'  # with double quotes, $ will be evaluated
git config --global icdiff.options "--highlight --line-numbers --numlines=3"
git config --global difftool.icdiff.cmd 'icdiff --highlight --line-numbers --numlines=3 $LOCAL $REMOTE'
```


##### Aliases

```bash
git config --global alias.st "status"
git config --global alias.cm "commit"
git config --global alias.ca "commit -am"
git config --global alias.cap '! f() { git commit -am "$@" && git push --set-upstream origin "$(git rev-parse --abbrev-ref HEAD)"; }; f'
git config --global alias.camend "commit --amend -am"
git config --global alias.amend "commit --amend --no-edit -a"
git config --global alias.br "branch"
git config --global alias.co "checkout"
git config --global alias.lg "log --graph --decorate --pretty=oneline --abbrev-commit"
git config --global alias.fp "fetch -p --all"  # purge and fetch all remotes
git config --global alias.df '! f() { git icdiff --color=always "$@" | less -eR; }; f'  # no FX (keep output in terminal)
git config --global alias.pr '! git push --set-upstream origin "$(git rev-parse --abbrev-ref HEAD)"'
git config --global alias.dm '! git fetch -p && for branch in `git branch -vv | grep '"': gone] ' | awk '"'{print $1}'"'"'`; do git branch -D $branch; done'  # 'delete merged' - local branches that have been deleted on remote
git config --global alias.gg '! f() { git checkout "${1:-master}" && git dm && git pull; }; f'
git config --global alias.pall '! f() {     START=$(git branch | grep "\*" | sed "s/^.//");     for i in $(git branch | sed "s/^.//"); do         git checkout $i;         git pull || break;     done;     git checkout $START; }; f'  # 'pull all' - pull local branches that have been updated on remote
```


### Get ready to install favourite apps from casks and Mac App Store (MAS)

```bash
brew install mas  # Please note that mas will not allow you to install (or even purchase) an app for the first time: it must already be in the Purchased tab of the App Store.
```


### Install purchased apps

```bash
# iStat Menus - bjango.com/mac/istatmenus
mas install 1319778037
# Magnet - magnet.crowdcafe.com
mas install 441258766
# DaisyDisk - daisydiskapp.com
mas install 411643860
```


### Install free apps

When launching apps for the first time, you might have to accept the dev under `System Preferences/Security & Privacy/General`
```bash
# Google Chrome - google.com/chrome
brew cask install google-chrome
# Amphetamine - roaringapps.com/app/amphetamine
mas install 937984704
# Telegram - macos.telegram.org
mas install 747648890
# Copyclip - fiplab.com/apps/copyclip-for-mac
mas install 595191960
# PIA VPN - privateinternetaccess.com - Requires manual install from ~/Library/Caches/Homebrew/downloads
brew cask install private-internet-access
# Tunnelblick OpenVPN - tunnelblick.net
brew cask install tunnelblick
# The Unarchiver - theunarchiver.com
brew cask install the-unarchiver
# f.lux - justgetflux.com
brew cask install flux
# VLC - videolan.org/vlc
brew cask install vlc
# Slack - slack.com
brew cask install slack
# Zoom.us - zoom.us
brew cask install zoomus
# Whatsapp - whatsapp.com
brew cask install whatsapp
# Dropbox - dropbox.com
brew cask install dropbox
# Authy - authy.com
brew cask install authy
# Docker - docker.com
brew cask install docker
```


### Install dev stuff

```bash
# Docker CE - docker.com/community-edition
brew cask install docker
brew install docker-compose
# get your favourite python versions - github.com/momo-lab/pyenv-install-latest
git clone https://github.com/momo-lab/pyenv-install-latest.git "$(pyenv root)"/plugins/pyenv-install-latest
git clone git://github.com/concordusapps/pyenv-implict.git "$(pyenv root)"/plugins/pyenv-implict
# list all available python versions
pyenv install -l
pyenv install-latest 2.7
pyenv install-latest 3.7
# Sublime Text - sublimetext.com
brew cask install sublime-text
# Sublime Merge - sublimemerge.com
brew cask install sublime-merge
```


##### [pyenv](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md#command-reference) and [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv#usage)

Note: pyvenv-virtualenv needs to be initialised in [`~/.bash_profile`](/.bash_profile), or in `~/.bashrc` if both files are [maintained separately](https://github.com/pyenv/pyenv-virtualenv/issues/36#issuecomment-48387008):

```sh
eval "$(pyenv init -)"
if which pyenv-virtualenv-init > /dev/null; then eval "$(pyenv virtualenv-init -)"; fi
```

```sh
pyenv global 3.7.7 2.7.15  # set default versions: prefer py3 over py2
source ~/.bash_profile  # global history etc
pip install --upgrade pip
# install virtualenv based on current pyenv Python version, inheriting installed packages
pyenv virtualenv --system-site-packages <venv-name>
# install virtualenv based on 3.6.8 pyenv Python version, inheriting installed packages
pyenv virtualenv 3.7.7 --system-site-packages <venv-name>
# activate, deactivate, delete
pyenv activate <venv-name>
pyenv deactivate
pyenv uninstall <venv-name>
```


### Kubernetes CLI ([kubectl](https://kubernetes.io/docs/reference/kubectl/))

Kubectl is a command line interface for running commands against Kubernetes clusters (like viewing logs or executing commands on pods).
See this [Kubectl Cheatsheet](https://gist.github.com/ddelange/24575a702a10c2cb6348c4c7f342e0eb) for plug & play bash functions (they are already in [`~/.bash_profile`](/.bash_profile)).


## Cleanup ([different options](https://github.com/Homebrew/brew/issues/3784#issuecomment-364675767))

```bash
brew cleanup --prune=0  # delete cache older than 0 days
```


## [Fancy Dropbox screen shot sharing](https://github.com/ddelange/mac-smart-bitly-shortcut)

AppleScript to shorten links using the bitly API in a smart way directly with a keyboard shortcut


## [Gigabit USB Driver OS X 10.9+](https://www.asix.com.tw/products.php?op=pItemdetail&PItemID=131;71;112)

For almost any [Gigabit Ethernet USB hub](https://www.ebay.com/itm/3-Ports-USB-3-0-Hub-Gigabit-Ethernet-Lan-RJ45-Network-Adapter-Hub-Hot-Lot-YT/183586523117)
- Unzip [`AX88179_178A_macintosh_Driver_Installer_v2.13.0.zip`](/AX88179_178A_macintosh_Driver_Installer_v2.13.0.zip)
- Install driver `pkg` from `AX88179_178A.dmg`
- Restart


## Misc

- To revert to the classic iTunes playlist view from before v12.6:
  - Open your iTunes library
  - Open and run [`Restore old iTunes playlists view.scpt`](/Restore%20old%20iTunes%20playlists%20view.scpt).
- New `Secrets.prefpane` for Mojave?
- shortcuts file: cmd L, fn+backspace, ⌘-⌥-V, lock, cmd space, screenshots, shortcut changing, cmd shift T, control up/down, cmd ~ or \` to switch windows, ~. to stop ssh, ⌘-^-space for emoji chooser
- Scroll horizontally using shift + mouse wheel
- iTunes Services, LaTeX opruimen, Dougs applescripts, [Show in Playlists](http://dougscripts.com/itunes/scripts/ss.php?sp=showinplaylists)
- [Fix playlist view](https://apple.stackexchange.com/a/316497/292695)
- low battery warning plist
- daisydisk