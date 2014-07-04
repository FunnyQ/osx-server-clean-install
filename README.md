# Mac mini 安裝 osx server 10.9 步驟記錄



## 重新安裝 Mac OSX

* 清除主硬碟，命名為 Server HD
* 重新安裝 OSX 10.9
* 設定好帳號與安全的密碼

## 到 Mac App Store 更新系統並下載重要軟體

* 更新至最新版本的 OSX
 * 下載安裝 [Server.app][1]
* 下載安裝 [Xcode.app][2]
* 下載安裝 [Dropbox.app][3]

## 環境設定

* 先開啟一次 Xcode.app 並依照指示同意使用規範
* 開啟 Server.app 並做基本設定

    * 概觀
        * 設定電腦名稱為 `<hostname>.com`
        * 更新主機名稱（要設定成 `<hostname>.com` 的話要注意 DNS 反解必須一致，否則使用建議選項即可）
    * 設定
        * ✔ 允許使用 SSH 從遠端登
        * ✔ 啟用螢幕共享與遠端管理
        * ✔ 啟用 Apple 推播通知
        * （反正全部 ✔ …）
    * 其他服務與進階設定可以等安裝完再依需求設定

* 將其他工作電腦的 rsa pub key 加入 `~/.ssh/authorized_keys`
* 編輯 `/etc/sshd_config`，設定下述項目，讓主機僅允許使用 pubkey ssh 登入

````sshd_config
PubkeyAuthentication yes
PasswordAuthentication no
ChallengeResponseAuthentication no
````

* 安裝 [Homebrew][4]

````install_homebrew
$ ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"
$ brew doctor #確認環境沒有問題
$ brew update
````

* 安裝 git

````install_git
$ brew install git
````

* 安裝 apple gcc

````apple-gcc
$ brew tap homebrew/dupes
$ brew install apple-gcc42
````

* 安裝 zsh

````
$ brew install zsh
````

* 安裝 [oh-my-zsh][5]

````
$ curl -L http://install.ohmyz.sh | sh
````

* 安裝 [z][6]

````
$ brew install z
````

* （把自己的 dotfiles 設定好）
* 安裝 [zsh-syntax-highlighting][7]

````
$ cd ~/.oh-my-zsh/custom/plugins
$ git clone git://github.com/zsh-users/zsh-syntax-highlighting.git
$ cd ~
````

* 安裝 [XQuartz][8]
* 安裝完後先登出再重新登入系統
* 安裝 Imagemagick

````imagemagic
$ brew install imagemagick
````

* 安裝 MySQL

````
$ brew install mysql
$ unset TMPDIR
$ mysql_install_db --verbose --user=`whoami` --basedir="$(brew --prefix mysql)" --datadir=/usr/local/var/mysql --tmpdir=/tmp
$ mysql.server start
$ mysqladmin -u root password ‘\<yourPassword\>’
$ mkdir -p ~/Library/LaunchAgents
$ find /usr/local/Cellar/mysql/ -name "homebrew.mxcl.mysql.plist" -exec cp ~/Library/LaunchAgents/ ;
$ launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
````

* 安裝 [RVM][9]

````
$ curl -L https://get.rvm.io | bash -s stable
$ . ~/.profile
$ source ~/.profile
````

* 在 RVM 安裝 Ruby 2.0

````
$ brew install libyaml
$ rvm pkg install openssl
$ rvm install 2.0.0 \\
--with-openssl-dir=$HOME/.rvm/usr \\
--verify-downloads 1
$ rvm use 2.0.0
````

* 安裝必要的 gems

````
$ gem install rails 
$ gem install mysql2
````

* 安裝 passenger

````
$ gem install passenger
$ xcode-select --install #這個不安裝下面 compile 會噴 error
$ passenger-install-apache2-module
````

[1]:    https://itunes.apple.com/tw/app/os-x-server/id714547929?l=zh&mt=12
[2]:    https://itunes.apple.com/tw/app/xcode/id497799835?l=zh&mt=12
[3]:    https://www.dropbox.com/
[4]:    http://brew.sh
[5]:    https://github.com/robbyrussell/oh-my-zsh
[6]:    https://github.com/rupa/z
[7]:    http://github.com/zsh-users/zsh-syntax-highlighting
[8]:    http://xquartz.macosforge.org/landing
[9]:    http://rvm.io