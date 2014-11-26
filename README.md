# OSX server 10.9 重新安裝步驟記錄

在 Mac Mini Server 或其它可運行 OSX 的伺服器上，可參考以下步驟來安裝 OSX Server 10.9。基本的 PHP、MySQL、Apache 以外，Ruby on Rails application 也可以 deploy 在安裝好的伺服器上運行（如 Redmine 或自行開發的 RoR 專案等）。並可透過 Apple 官方的 Server.app 來做大部分的伺服器管理與設定，沒有太大的問題。

## 重新安裝 Mac OSX

* 清除主硬碟，命名為 Server HD
* 重新安裝 OSX 10.9
* 設定好帳號與安全的密碼

## 到 Mac App Store 更新系統並下載重要軟體

* 更新至最新版本的 OSX
* 下載安裝 [Server.app][1]
* 下載安裝 [Xcode.app][2]
* 下載安裝 [Dropbox.app][3]

## 基本環境設定

* 先開啟一次 Xcode.app 並依照指示同意使用規範
* 開啟 Server.app 並做基本設定

    * 概觀
        * 設定電腦名稱為 `<yourHostname>.com`
        * 更新主機名稱（要設定成 `<yourHostname>.com` 的話要注意 DNS 反解必須一致，否則使用建議選項即可）
    * 設定
        * ✔ 允許使用 SSH 從遠端登入
        * ✔ 啟用螢幕共享與遠端管理
        * ✔ 啟用 Apple 推播通知
        * （反正全部 ✔ …）
    * 其他服務與進階設定可以等安裝完再依需求設定

* 生成 rsa key 與 authorized_keys

````
$ ssh-keygen -t rsa
$ touch ~/.ssh/authorized_keys
````

* 將其他工作電腦的 `id_rsa.pub` 內容寫入 `~/.ssh/authorized_keys`
* 將 Server 的 `id_rsa.pub` 內容加入 Github, bitbucket 等 project Deploy Key
* 編輯 `/etc/sshd_config`，設定下述項目，讓主機僅允許使用 pubkey ssh 登入

````sshd_config
PubkeyAuthentication yes
PasswordAuthentication no
ChallengeResponseAuthentication no
````

* 安裝 [Homebrew][4]

````install_homebrew
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
$ brew doctor
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

* 安裝 [zsh-syntax-highlighting][7]

````
$ cd ~/.oh-my-zsh/custom/plugins
$ git clone git://github.com/zsh-users/zsh-syntax-highlighting.git
$ cd ~
````

（最後記得要先把自己的 dotfiles 設定好，讓一切正常。例如`.zshrc`、`.gitconfig`、`.gitignore_global`等等）

## 建立可以運行 Ruby on Rails app 的環境

* 安裝 [XQuartz][8]
* 安裝完後先登出再重新登入系統
* 安裝 Imagemagick

````imagemagic
$ brew install imagemagick
````

* 安裝 MySQL

````sh
$ brew install mysql
$ unset TMPDIR
$ mysql_install_db --verbose --user=`whoami` --basedir="$(brew --prefix mysql)" --datadir=/usr/local/var/mysql --tmpdir=/tmp
$ mysql.server start
$ mysql_secure_installation
$ mkdir -p ~/Library/LaunchAgents

# 10.10 Yosemite 用下面這個指令
$ ln -sfv /usr/local/opt/mysql/*plist ~/Library/LaunchAgents

# 10.9 以及更早的版本用下面這個指令
# $ find /usr/local/Cellar/mysql/ -name "homebrew.mxcl.mysql.plist" -exec cp ~/Library/LaunchAgents/ ;
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
$ rvm install 2.1.5 \
--with-openssl-dir=$HOME/.rvm/usr \
--verify-downloads 1
$ rvm --default use 2.1.5
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

* 設定 passenger

````
$ sudo touch /Library/Server/Web/Config/apache2/other/passenger.conf
````

在 `passenger.conf` 裡加入安裝 apache2-module 時提示你的設定，大致上會長得像下面這樣

````
LoadModule passenger_module /Users/[YourUserName]/.rvm/gems/ruby-2.0.0-p481/gems/passenger-4.0.45/buildout/apache2/mod_passenger.so
   <IfModule mod_passenger.c>
     PassengerRoot /Users/[YourUserName]/.rvm/gems/ruby-2.0.0-p481/gems/passenger-4.0.45
     PassengerDefaultRuby /Users/[YourUserName]/.rvm/gems/ruby-2.0.0-p481/wrappers/ruby
   </IfModule>
````

* 重啟 Server
* Done!

---- 

完成後建議做個備份的映像檔，如果日後發生問題的話可以快速恢復基本的伺服器運作環境。

----

### Reference

   1. https://github.com/rocodev/guides/wiki
   2. http://www.redmine.org/projects/redmine/wiki/RedmineInstallOSXMavericksServer

[1]:    https://itunes.apple.com/tw/app/os-x-server/id714547929?l=zh&mt=12
[2]:    https://itunes.apple.com/tw/app/xcode/id497799835?l=zh&mt=12
[3]:    https://www.dropbox.com/
[4]:    http://brew.sh
[5]:    https://github.com/robbyrussell/oh-my-zsh
[6]:    https://github.com/rupa/z
[7]:    http://github.com/zsh-users/zsh-syntax-highlighting
[8]:    http://xquartz.macosforge.org/landing
[9]:    http://rvm.io
