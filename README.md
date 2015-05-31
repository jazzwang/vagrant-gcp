# vagrant-gcp

這裡存放了一些關於如何使用 `vagrant-google` 擴充元件來佈署 Hadoop BigTop 與 Cloudera Manager 到 Google Compute Engine 的範例。

[TOC]

## Requirement 前置作業

 * 註冊一個 Google Cloud Platform 的帳號
    - https://console.developers.google.com/billing/freetrial - 註冊試用。
    - 產生 Google Cloud Platform API 的 OAuth P12 金鑰
    - 啟用 Google Compute Engine 的計費
 * 註冊一個 Koding 的帳號
    - https://koding.com/Register - 註冊試用
    - 預設有一台 Ubuntu 的虛擬機器可以免費使用。

## STEP 1 : 啟動 VM 並複製 vagrant-gcp 範例

* 首先，登入 Koding.com，並開啟 VM
* 接著，在 Terminal 中使用 git 將本範例複製到虛擬機器中
```
jazzwang: ~ $ git clone http://github.com/jazzwang/vagrant-gcp
Cloning into 'vagrant-gcp'...
remote: Counting objects: 96, done.
remote: Compressing objects: 100% (50/50), done.
remote: Total 96 (delta 19), reused 0 (delta 0), pack-reused 45
Unpacking objects: 100% (96/96), done.
Checking connectivity... done.
```
* 接著，我們先切換到 vagrant-gcp 目錄，並複製 `env_sample` 到 `.env` 。後續的步驟會需要將 Google Cloud Platform API 的 OAuth 設定填寫到 `.env` 檔中。
```
jazzwang: ~ $ cd vagrant-gcp/
jazzwang: ~/vagrant-gcp $ cp env_sample .env
```

[TOC]

## STEP 2 : 安裝 Google Cloud SDK

* 透過這個範例，您可以快速地體驗一下 Google Cloud Platform 的功能。為了方便後續範例的操作，我們使用 Google Cloud SDK 來快速打造一個 Google Cloud Platform 的實驗環境。
* 為了簡化步驟，本範例內含一個安裝 Google Cloud SDK 的腳本，您只需要執行 `bin/install-gcloud` 並遵循畫面的指示，即可設定好一個能透過指令執行 Google Compute Engine 的實驗環境。
* 以下是在 Koding.com 的 VM 中，執行的示範過程。首先，它會詢問您安裝的路徑，預設為您的家目錄。若不修改，請按 Enter 繼續。

```
jazzwang: ~/vagrant-gcp $ bin/install-gcloud 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   425    0   425    0     0   2395      0 --:--:-- --:--:-- --:--:--  2401
Downloading Google Cloud SDK install script: https://dl.google.com/dl/cloudsdk/release/install_google_cloud_sdk.bash
######################################################################## 100.0%
Running install script from: /tmp/tmp.JRkDJv8o67/install_google_cloud_sdk.bash
curl -# -f https://dl.google.com/dl/cloudsdk/release/google-cloud-sdk.tar.gz
######################################################################## 100.0%
 
Directory to extract under (this will create a directory google-cloud-sdk) (/home/jazzwang):  
```

 * 接下來，您會看到安裝程式在解開壓縮檔到指定的安裝路徑。第二個要回答的問題是「您是否願意提供匿名的使用資料，來幫助 GOogle Cloud Platform 改善這個 SDK ？」。預設為 Yes，若您不想提供匿名資料，請按 `n`。 

```
tar -C /home/jazzwang -xvf /tmp/tmp.j30yImoWzP/google-cloud-sdk.tar.gz
google-cloud-sdk/
google-cloud-sdk/help/
google-cloud-sdk/help/text.long/
google-cloud-sdk/help/text.long/gcloud_version
google-cloud-sdk/help/text.long/gcloud_init
google-cloud-sdk/help/text.long/gcloud_info
google-cloud-sdk/help/text.long/gcloud_help
google-cloud-sdk/help/text.long/gcloud
google-cloud-sdk/.install/.download/

.... SKIP ....... SKIP ....... SKIP ....... SKIP ...

/home/jazzwang/google-cloud-sdk/install.sh
Welcome to the Google Cloud SDK!
 
To help improve the quality of this product, we collect anonymized data on how
the SDK is used. You may choose to opt out of this collection now (by choosing
'N' at the below prompt), or at any time in the future by running the following
command:
    gcloud config set --scope=user disable_usage_reporting true
 
Do you want to help improve the Google Cloud SDK (Y/n)?
```

* 第三個問題是「您是否要修改環境變數 $PATH 並啟用 Bash 的自動補字功能？」，預設為 Yes。通常打開會比較方便，這裡建議按 `Enter` 繼續。

```
This will install all the core command line tools necessary for working with
the Google Cloud Platform.
 
 
The following components will be installed:
    -------------------------------------------------------------------------------------------
    | BigQuery Command Line Tool                                        |     2.0.18 | < 1 MB |
    | BigQuery Command Line Tool (Platform Specific)                    |     2.0.18 | < 1 MB |
    | Cloud DNS Admin Command Line Interface                            | 2015.05.19 | < 1 MB |
    | Cloud SDK Core Command Line Tools                                 |          1 |        |
    | Cloud SDK Core Libraries (Platform Specific)                      | 2014.10.20 | < 1 MB |
    | Cloud SQL Admin Command Line Interface                            | 2015.05.06 | < 1 MB |

.... SKIP ....... SKIP ....... SKIP ....... SKIP ...

|- Installing: kubectl (Linux, x86_64)                      -|
|============================================================|
 
Creating backup and activating new installation...
 
Update done!
 
Modify profile to update your $PATH and enable bash completion? (Y/n)?
```

* 乘上一個問題，若您回答 Yes，那麼下一個問題是「您存放環境變數的 rc 檔位置」。通常預設都是 .bashrc，因此這個問題也可以按下 `Enter` 鍵繼續。

```
The Google Cloud SDK installer will now prompt you to update an rc 
file to bring the Google Cloud CLIs into your environment.
 
Enter path to an rc file to update, or leave blank to use 
[/home/jazzwang/.bashrc]:
```

* 以上幾個問題回答完畢，您會看到一段提示說明，告知您接下來需要做的動作。

```
Backing up [/home/jazzwang/.bashrc] to [/home/jazzwang/.bashrc.backup].
[/home/jazzwang/.bashrc] has been updated.
Start a new shell for the changes to take effect.
 
For more information on how to get started, please visit:
  https://developers.google.com/cloud/sdk/gettingstarted
 
 Authenticating to Google Compute Engine by running:
 
 $ source ~/.bashrc ; gcloud auth login
 $ gcloud config set project PROJECT
 $ gcloud compute config-ssh
 
```

[TOC]

## STEP 3 : 設定 Google Cloud SDK

* 接下來，我們需要進一步設定 Google Cloud SDK。請參考上一步的指示，輸入指令 `source ~/.bashrc ; gcloud auth login` 來將登入 Google Cloud Platform 的認証資訊，存放於 VM 的開發環境中。
* 輸入上述指令後，會看到畫面上出現一串超連結，請點選超連結，並登入您申請 Google Cloud Platform 的帳號密碼。並將畫面上出現的認証碼複製貼上到 `Enter verification code:` 後方。

```
jazzwang: ~/vagrant-gcp $ source ~/.bashrc ; gcloud auth login
Go to the following link in your browser:
 
    https://accounts.google.com/o/oauth2/auth?redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&prompt=select_account&response_type
=code&client_id=32555940559.apps.googleusercontent.com&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email+https%3A%2F%
2Fwww.googleapis.com%2Fauth%2Fcloud-platform+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fappengine.admin+https%3A%2F%2Fwww.googleapi
s.com%2Fauth%2Fcompute&access_type=offline
  
Enter verification code:  
```

* 接著，請遵照畫面上的指示，繼續輸入 `gcloud config set project PROJECT_ID`，請注意，這裡的 `PROJECT_ID` 是您申請的 Google Cloud Platform 的專案代號。預設專案名稱為 "My Project"，請注意這裡要填的是「專案ID（Project ID)」，不是專案名稱，請自行依照您取得的「專案 ID」進行命令的微調。

```
Saved Application Default Credentials.
 
You are now logged in as [************@gmail.com].
Your current project is [None].  You can change this setting by running:
  $ gcloud config set project PROJECT_ID

jazzwang: ~/vagrant-gcp $ gcloud config set project  valued-aquifer-528
```

* 接下來，由於我們會需要一組 SSH 金鑰，可以用來登入 Google Compute Engine 的 VM，因此您可以使用 `gcloud compute config-ssh` 指令來產生預設的金鑰，並讓 Google Cloud SDK 自動幫您上傳到 Google Cloud Platform。

```
jazzwang: ~/vagrant-gcp $ gcloud compute config-ssh
WARNING: You do not have an SSH key for Google Compute Engine.
WARNING: [/usr/bin/ssh-keygen] will be executed to generate a key.
This tool needs to create the directory [/home/jazzwang/.ssh] before 
being able to generate SSH keys.
 
Do you want to continue (Y/n)?
```

* 按下 `Enter` 鍵，讓 Google Cloud SDK 自動幫您產生到家目錄下預設存放 SSH 金鑰的路徑。如果您希望未來在使用這組金鑰時，需要另外打密碼，請在以下看到的畫面處輸入您想要的保護密碼。這裡為了後續操作方便，並沒有輸入保護密碼。直接按 `Enter` 跳過。產出的私鑰預設存放於 `${HOME}/.ssh/google_compute_engine`，公鑰則存放於 `${HOME}/.ssh/google_compute_engine.pub`。

```
WARNING: You do not have an SSH key for Google Compute Engine.
WARNING: [/usr/bin/ssh-keygen] will be executed to generate a key.
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/jazzwang/.ssh/google_compute_engine.
Your public key has been saved in /home/jazzwang/.ssh/google_compute_engine.pub.
The key fingerprint is:
e4:c7:d5:48:c7:4a:11:4d:a9:0b:40:be:dd:c7:69:12 jazzwang@jazzwang
The key's randomart image is:
+--[ RSA 2048]----+
|       ..   +*o. |
|       ..  ..++  |
|        o. .Eo.  |
|       o +.ooo . |
|        S +.o.=  |
|         .  .+   |
|                 |
|                 |
|                 |
+-----------------+
Updated [https://www.googleapis.com/compute/v1/projects/valued-aquifer-528].
WARNING: No host aliases were added to your SSH configs because you do not have any instances. Try running this command again afte
r creating some instances.
```

* **備註：**如果您遇到以下的錯誤訊息，代表您填錯 `PROJECT_ID` 或者尚未啟用「Google Compute Engine 的計費」，請回到 https://console.developers.google.com/ 去開啟您的 Google Compute Engine 服務。

```
ERROR: (gcloud.compute.config-ssh) Could not fetch project resource:
 - The resource 'projects/hadoop-labs' was not found
```

[TOC]

## STEP 4 : 安裝 Vagrant 與相關套件

* Vagrant　是一套用來管理虛擬機器的軟體，預設用來管理運行於 VirtualBox, VMWare 等私有雲的虛擬機器。透過安裝擴充元件的方式，可以額外支援像是 Amazon AWS EC2 與 Google Compute Engine 等公有雲的虛擬機器。
* 本範例預計會使用到三個 Vagrant 的擴充元件(Plugin)，依序為：
	* [vagrant-google](https://github.com/mitchellh/vagrant-google) - 讓 Vagrant 支援管理 Google Compute Engine 的能力
	* [vagrant-env](https://github.com/gosuri/vagrant-env) - 讓 Vagrant 支援從 .env 檔讀取環境變數的能力
	* [vagrant-hostmanager](https://github.com/smdahlen/vagrant-hostmanager) - 讓 Vagrant 支援自動產生並同步多台虛擬機器 /etc/hosts 的能力

* 為了簡化安裝設定 Vagrant 的流程，本範例提供了 `install-vagrant` 的腳本。請執行 `bin/install-vagrant` 就可以依序完成安裝 (A) Vagrant (B) vagrant-google (C) vagrant-env (D) vagrant-hostmanager (E) 新增 vagrant-google 的預設 Vagrant Box

```
jazzwang: ~/vagrant-gcp $ bin/install-vagrant
Ign http://ap-southeast-1.ec2.archive.ubuntu.com trusty InRelease
Ign http://ap-southeast-1.ec2.archive.ubuntu.com trusty-updates InRelease
Hit http://ap-southeast-1.ec2.archive.ubuntu.com trusty Release.gpg
Get:1 http://ap-southeast-1.ec2.archive.ubuntu.com trusty-updates Release.gpg [933 B]

.... SKIP ....... SKIP ....... SKIP ....... SKIP ...

Unpacking vagrant (1:1.7.2) ...
Setting up vagrant (1:1.7.2) ...
Installing the 'vagrant-hostmanager' plugin. This can take a few minutes...
Installed the plugin 'vagrant-hostmanager (1.5.0)'!
Installing the 'vagrant-google' plugin. This can take a few minutes...
Installed the plugin 'vagrant-google (0.1.5)'!
Installing the 'vagrant-env' plugin. This can take a few minutes...
Installed the plugin 'vagrant-env (0.0.2)'!
==> box: Adding box 'gce' (v0) for provider: 
    box: Downloading: https://github.com/mitchellh/vagrant-google/raw/master/google.box
==> box: Successfully added box 'gce' (v0) for 'google'!
```

* 如果想瞭解目前所使用的 Vagrant 擴充元件有哪些，可以執行 `vagrant plugin list` 來檢查，若自動安裝正確，您應該會看到以下結果：

```
jazzwang: ~/vagrant-gcp $ vagrant plugin list
vagrant-env (0.0.2)
vagrant-google (0.1.5)
vagrant-hostmanager (1.5.0)
vagrant-share (1.1.3, system)
```

* **備註：**如果您看到的擴充元件列表與上述不同，也可以重新執行以下指令，來確保擴充元件已正確安裝：

```
jazzwang: ~/vagrant-gcp $ vagrant plugin install --verbose vagrant-hostmanager
jazzwang: ~/vagrant-gcp $ vagrant plugin install --verbose vagrant-google
jazzwang: ~/vagrant-gcp $ vagrant plugin install --verbose vagrant-env
```

[TOC]

## STEP 5 : 設定 .env 環境變數檔

* 接下來，我們將使用 Vagrant 來控制 Google Compute Engine 啟動、關閉虛擬機器的流程。為了讓各位可以做最小幅度的修改，我們在 Vagrantfile 中使用環境變數來讀取 Google Compute Engine 的 OAuth P12 金鑰與您專屬的服務帳戶電子郵件地址。
* 若您還沒有產生 Google Cloud Platform 的 OAuth P12 金鑰，請先連線至https://console.developers.google.com/project/＄{PROJECT_ID}/apiui/credential (請自行使用您申請到的專案代號來取代 ${PROJECT_ID} 處的字串，或者在 Google Developer Console 選擇「API和驗證 > 憑證 > OAuth > 建立新的用戶端ID > 應用程式類型 : 服務帳戶 > 索引鍵類型
: P12 金鑰 > 建立用戶端 ID」)
* 儲存 P12 金鑰檔，並且以拖曳的方式，上傳至 Koding.com 的 VM 中
* 於 Koding.com 的 VM 中，建立一隱藏目錄，名為 `.gcp`

```
jazzwang: ~/vagrant-gcp $ mkdir -p ~/.gcp
```

* 將上傳的 P12 金鑰檔放入家目錄下的 `.gcp` 子目錄中。
* 通常 P12 金鑰檔的名稱為 ${PROJECT_NAME}-000000000000.p12 ，是使用「專案名稱(Project Name)」開頭，後面接著共計 12 碼的數字。

```
GOOGLE_PROJECT_ID="這裡填入 PROJECT_ID"
GOOGLE_CLIENT_EMAIL="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx@developer.gserviceaccount.com"
GOOGLE_KEY_LOCATION="~/.gcp/********-000000000000.p12"
GOOGLE_SSH_USER=$USER
GOOGLE_SSH_KEY_LOCATION="~/.ssh/google_compute_engine"
```

* 這裡需要修改的內容包括 (A) GOOGLE_PROJECT_ID 環境變數 (B) GOOGLE_CLIENT_EMAIL 環境變數  (C) GOOGLE_KEY_LOCATION 環境變數 ，共計三處。
	* 關於 GOOGLE_PROJECT_ID 同上說明，請參考 https://console.developers.google.com/ 的專案資訊，正確填寫專案代號（注意！不是專案名稱哦！）
	* 關於 GOOGLE_CLIENT_EMAIL 請參考 https://console.developers.google.com/project/＄{PROJECT_ID}/apiui/credential 的內容，填入「OAuth >  服務帳戶 > 電子郵件地址」的內容。
	* 關於 GOOGLE_CLIENT_EMAIL 請參考 https://console.developers.google.com/project/＄{PROJECT_ID}/apiui/credential 的內容，填入「OAuth >  服務帳戶 > 電子郵件地址」的內容。
	* 關於 GOOGLE_KEY_LOCATION 請依您上傳到 VM 的 P12 金鑰檔的檔案名稱加以修正。
* 至於 GOOGLE_SSH_USER 環境變數與 GOOGLE_SSH_KEY_LOCATION 環境變數，可不做修改。因為 $USER 等於您目前所使用的「Koding.com VM 使用者名稱」。而 `~/.ssh/google_compute_engine` 則是在 STEP 3 產生的 SSH 金鑰。若您要修改成自己的 SSH 金鑰 (Ex. ~/.ssh/id_rsa)，您需要使用 Google Cloud SDK 註冊該 SSH 金鑰，會比較麻煩。
* **注意！** 建議若是在自己的環境中執行時，請不要使用 root 身份，否則有些虛擬機器的映像檔(VM Image)會限制 root 的 SSH 連線。會造成 Vagrant 正確開啟了虛擬機器，但遲遲無法連線的狀態。
* **備註：**有用過 Vagrant 控制 VirtualBox 的管理者可能會注意到觀念上不太一樣的地方，預設 Vagrant 會使用 `vagrant` 這個使用者身份連線到開啟的虛擬機器中，然後透過 sudo 權限去執行佈署用 (Provision) 的程式碼。但在 Google Compute Engine 上，則是使用 GOOGLE_SSH_USER 環境變數所指定的使用者名稱作為預設 SSH 登入的身份。因此，請注意 GOOGLE_SSH_USER 環境變數設定的使用者名稱，必須

[TOC]

## Lab 0 : 驗證 Vagrant 可否啟動 GCE VM

* 經過 STEP 1 ~ 5 的設定後，您總算可以透過簡單的指令，透過 Vagrantfile 與命令列彈性地控制 Google Compute Engine 上的虛擬機器了。
* 本範例第一個實作範例是在 Google Compute Engine 上啟動一台 Debian 的虛擬機器。請切換到 `0_quickstart` 子目錄，並執行指令 `vagrant status` 進行檢查 。若 STEP 1~5 的設定都正確，您應該會看到以下的結果：

```
jazzwang: ~/vagrant-gcp $ cd 0_quickstart/
jazzwang: ~/vagrant-gcp/0_quickstart $ vagrant status
Current machine states:

default                   not created (google)

The Google instance is not created. Run `vagrant up` to create it.
```
* 若您看到如下的錯誤訊息，通常代表 .env 檔設定有誤。(以下範例是 P12 金鑰檔未能正確填寫造成的錯誤訊息)

```
jazzwang: ~/vagrant-gcp/0_quickstart $ vagrant status
/home/jazzwang/.vagrant.d/gems/gems/google-api-client-0.8.6/lib/google/api_client/auth/key_utils.rb:88:in `rescue in load_key': In
valid keyfile or passphrase (ArgumentError)
        from /home/jazzwang/.vagrant.d/gems/gems/google-api-client-0.8.6/lib/google/api_client/auth/key_utils.rb:80:in `load_key'

.... SKIP ....... SKIP ....... SKIP ....... SKIP ...
```

＊ 若您可正確看到 `vagrant status` 的結果，顯示虛擬機器的狀態是 `not created (google)` ，那代表您的設定是正確的。請繼續執行指令 `vagrant up` ，就可以在 Google Compute Engine 上建立一台 Debian 的虛擬機器。執行結果如以下範例：

```
jazzwang: ~/vagrant-gcp/0_quickstart $ vagrant up
Bringing machine 'default' up with 'google' provider...
==> default: Warning! The Google provider doesn't support any of the Vagrant
==> default: high-level network configurations (`config.vm.network`). They
==> default: will be silently ignored.
==> default: Launching an instance with the following settings...
==> default:  -- Name:            quickstart
==> default:  -- Type:            n1-standard-1
==> default:  -- Disk type:       pd-standard
==> default:  -- Disk size:       10 GB
==> default:  -- Disk name:       
==> default:  -- Image:           debian-7-wheezy-v20140619
==> default:  -- Zone:            asia-east1-a
==> default:  -- Network:         default
==> default:  -- Metadata:        '{}'
==> default:  -- Tags:            '[]'
==> default:  -- IP Forward:      
==> default:  -- External IP:     
==> default:  -- Autodelete Disk: true
==> default: Waiting for instance to become "ready"...
==> default: Machine is booted and ready for use!
==> default: Waiting for SSH to become available...
==> default: Machine is ready for SSH access!
==> default: Rsyncing folder: /home/jazzwang/vagrant-gcp/0_quickstart/ => /vagrant
```

* **備註：**您若看到以下訊息，代表啟動所需的預設 Vagrant Box 'gce' 不存在，請重新執行指令 `vagrant box add gce https://github.com/mitchellh/vagrant-google/raw/master/google.box` 進行下載。並使用 `vagrant box list` 指令檢查，是否有名為 `gce` 的 Vagrant Box (虛擬機器映像檔，VM Image)存在。

```
jazzwang: ~/vagrant-gcp/0_quickstart $ vagrant up
Bringing machine 'default' up with 'google' provider...
==> default: Box 'gce' could not be found. Attempting to find and install...
    default: Box Provider: google
    default: Box Version: >= 0
==> default: Adding box 'gce' (v0) for provider: google
    default: Downloading: gce
An error occurred while downloading the remote file. The error 
message, if any, is reproduced below. Please fix this error and try
again.
 
Couldn't open file /home/jazzwang/vagrant-gcp/0_quickstart/gce
jazzwang: ~/vagrant-gcp $ vagrant box add gce https://github.com/mitchellh/vagrant-google/raw/master/google.box
==> box: Adding box 'gce' (v0) for provider: 
    box: Downloading: https://github.com/mitchellh/vagrant-google/raw/master/google.box
==> box: Successfully added box 'gce' (v0) for 'google'!
jazzwang: ~/vagrant-gcp $ vagrant box list
gce (google, 0)
```

* 若您可以看到正確啟動的結果，接下來就可以使用 `vagrant ssh` 指令，連線進入 Google Compute Engine 上的虛擬機器。

```
jazzwang: ~/vagrant-gcp/0_quickstart $ vagrant ssh
Linux quickstart 3.2.0-4-amd64 #1 SMP Debian 3.2.57-3+deb7u2 x86_64
 
The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.
 
Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
jazzwang@quickstart:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Debian
Description:    Debian GNU/Linux 7.5 (wheezy)
Release:        7.5
Codename:       wheezy
jazzwang@quickstart:~$ exit
logout
Connection to 104.155.206.246 closed.
```

* 此時，您也可以用 Google Cloud SDK 的指令 `gcloud compute instances list` 來檢查目前啟動的虛擬機器狀態。

```
jazzwang: ~/vagrant-gcp/0_quickstart $ gcloud compute instances list
NAME       ZONE         MACHINE_TYPE  PREEMPTIBLE INTERNAL_IP   EXTERNAL_IP     STATUS
quickstart asia-east1-a n1-standard-1             10.240.98.137 104.155.206.246 RUNNING
```

* 由於 Google Compute Engine 使依照虛擬機器的執行時間計費，因此若您不想再使用了，可以使用 `vagrant destroy -f` 指令，將該虛擬機器強制關閉並刪除。

```
jazzwang: ~/vagrant-gcp/0_quickstart $ vagrant destroy -f
==> default: Terminating the instance...
jazzwang: ~/vagrant-gcp/0_quickstart $ gcloud compute instances list
NAME ZONE MACHINE_TYPE PREEMPTIBLE INTERNAL_IP EXTERNAL_IP STATUS
```

[TOC]

## Lab 1-1 : 佈署單機 Apache BigTop 於 CentOS 虛擬機器

* 第二個範例是關於使用 Vagrant 在 Google Compute Engine 上啟動一台 CentOS 6 的虛擬機器，然後透過 SSH 連線，手動執行佈署 provision.sh 的佈署腳本。這個腳本可以在 CentOS 6 上安裝 Apache BigTop 0.7 的 Hadoop, HBase, Pig 與 Hive 執行環境。
* 請切換到 `1_single_node_bigtop_centos6` 子目錄，並執行 `vagrant up` 指令。

```
jazzwang: ~/vagrant-gcp/0_quickstart $ cd ~/vagrant-gcp/1_single_node_bigtop_centos6/
jazzwang: ~/vagrant-gcp/1_single_node_bigtop_centos6 $ vagrant status
Current machine states:
 
default                   not created (google)
 
The Google instance is not created. Run `vagrant up` to create it.
jazzwang: ~/vagrant-gcp/1_single_node_bigtop_centos6 $ vagrant up
Bringing machine 'default' up with 'google' provider...
==> default: Warning! The Google provider doesn't support any of the Vagrant
==> default: high-level network configurations (`config.vm.network`). They
==> default: will be silently ignored.
==> default: Launching an instance with the following settings...
==> default:  -- Name:            bigtop1
==> default:  -- Type:            n1-standard-1
==> default:  -- Disk type:       pd-standard
==> default:  -- Disk size:       40 GB
==> default:  -- Disk name:       bigtop1
==> default:  -- Image:           centos-6-v20150423
==> default:  -- Zone:            asia-east1-a
==> default:  -- Network:         default
==> default:  -- Metadata:        '{}'
==> default:  -- Tags:            '[]'
==> default:  -- IP Forward:      
==> default:  -- External IP:     
==> default:  -- Autodelete Disk: true
==> default: Waiting for instance to become "ready"...
==> default: Machine is booted and ready for use!
==> default: Waiting for SSH to become available...
==> default: Machine is ready for SSH access!
==> default: Rsyncing folder: /home/jazzwang/vagrant-gcp/1_single_node_bigtop_centos6/ => /vagrant
jazzwang: ~/vagrant-gcp/1_single_node_bigtop_centos6 $ vagrant status
Current machine states:
 
default                   running (google)
 
The Google instance is running. To stop this machine, you can run
`vagrant halt`. To destroy the machine, you can run `vagrant destroy`.
```

* 虛擬機器創建完畢後，vagrant-google 擴充元件會用 rsync 將本地端的資料，複製到 Google Compute Engine 的虛擬機器上，並存放於 /vagrant 目錄中。
* 因此，我們想要執行的 `provision.sh` 因為 Vagrant 已將幫我們複製到 GCE VM 上了，您只需要 SSH 進入虛擬機器，然後以 sudo 權限執行 `/vagrant/provision.sh` 即可。

```
jazzwang: ~/vagrant-gcp/1_single_node_bigtop_centos6 $ vagrant ssh     
[jazzwang@bigtop1 ~]$ sudo /vagrant/provision.sh 
USER=root
Loaded plugins: downloadonly, fastestmirror, security
Setting up Install Process
Determining fastest mirrors
 * base: centos.mirrors.tds.net
 * extras: mirror.san.fastserv.com
 * updates: mirrors.cat.pdx.edu

.... SKIP ....... SKIP ....... SKIP ....... SKIP ...
```

* **備註：**由於 Google Compute Engine 一段時間就會更新虛擬機器的映像檔，因此如果未來各位在執行時無法正確執行，也有可能是因為 VM Image 的名稱有變動。此時，請先使用 Google Cloud SDK 的指令 `gcloud compute images list` 查詢目前 Google Cloud Engine 的映像檔名稱，然後才對應修改 Vagrantfile 內容的 google.image 變數內容。

```
jazzwang: ~/vagrant-gcp/1_single_node_bigtop_centos6 $ gcloud compute images list
NAME                                PROJECT           ALIAS              DEPRECATED STATUS
centos-6-v20150526                  centos-cloud      centos-6                      READY
centos-7-v20150526                  centos-cloud      centos-7                      READY
coreos-alpha-695-0-0-v20150528      coreos-cloud                                    READY
coreos-beta-681-0-0-v20150527       coreos-cloud                                    READY
coreos-stable-647-2-0-v20150528     coreos-cloud      coreos                        READY
backports-debian-7-wheezy-v20150526 debian-cloud      debian-7-backports            READY
debian-7-wheezy-v20150526           debian-cloud      debian-7                      READY
container-vm-v20150129              google-containers container-vm                  READY
container-vm-v20150305              google-containers container-vm                  READY
container-vm-v20150317              google-containers container-vm                  READY
container-vm-v20150505              google-containers container-vm                  READY
opensuse-13-1-v20150515             opensuse-cloud    opensuse-13                   READY
opensuse-13-2-v20150511             opensuse-cloud    opensuse-13                   READY
rhel-6-v20150526                    rhel-cloud        rhel-6                        READY
rhel-7-v20150526                    rhel-cloud        rhel-7                        READY
sles-11-sp3-v20150511               suse-cloud        sles-11                       READY
sles-12-v20150511                   suse-cloud        sles-12                       READY
ubuntu-1204-precise-v20150316       ubuntu-os-cloud   ubuntu-12-04                  READY
ubuntu-1404-trusty-v20150316        ubuntu-os-cloud   ubuntu-14-04                  READY
ubuntu-1410-utopic-v20150509        ubuntu-os-cloud   ubuntu-14-10                  READY
ubuntu-1504-vivid-v20150422         ubuntu-os-cloud   ubuntu-15-04                  READY
windows-server-2008-r2-dc-v20150331 windows-cloud     windows-2008-r2               READY
windows-server-2012-r2-dc-v20150331 windows-cloud     windows-2012-r2               READY
```

* 實驗結束後，**別忘了關閉 Google Compute Engine 的虛擬機器喔**～以免試用的額度用完後，開始正式從信用卡扣款。

```
jazzwang: ~/vagrant-gcp/1_single_node_bigtop_centos6 $ vagrant destroy -f
==> default: Terminating the instance...
```

[TOC]

## Lab 1-2 : 自動佈署 BigTop 於 Ubuntu 虛擬機器

* 第二個範例是關於使用 Vagrant 在 Google Compute Engine 上啟動一台 Ubuntu 14.04 的虛擬機器，並自動執行佈署腳本 `provision.sh`。這個腳本可以在 Ubuntu 14.04 上安裝 Apache BigTop 0.7 的 Hadoop, HBase, Pig 與 Hive 執行環境。
* 請切換到 `1_single_node_bigtop_ubuntu1404` 子目錄，並執行 `vagrant up` 指令。

```
jazzwang: ~/vagrant-gcp/1_single_node_bigtop_centos6 $ cd ~/vagrant-gcp/1_single_node_bigtop_ubuntu1404/
jazzwang: ~/vagrant-gcp/1_single_node_bigtop_ubuntu1404 $ vagrant status
Current machine states:
 
default                   not created (google)
 
The Google instance is not created. Run `vagrant up` to create it.
jazzwang: ~/vagrant-gcp/1_single_node_bigtop_ubuntu1404 $ vagrant up    
Bringing machine 'default' up with 'google' provider...
==> default: Warning! The Google provider doesn't support any of the Vagrant
==> default: high-level network configurations (`config.vm.network`). They
==> default: will be silently ignored.
==> default: Launching an instance with the following settings...
==> default:  -- Name:            bigtop2
==> default:  -- Type:            n1-standard-1
==> default:  -- Disk type:       pd-standard
==> default:  -- Disk size:       40 GB
==> default:  -- Disk name:       bigtop2
==> default:  -- Image:           ubuntu-1404-trusty-v20150316
==> default:  -- Zone:            asia-east1-a
==> default:  -- Network:         default
==> default:  -- Metadata:        '{}'
==> default:  -- Tags:            '[]'
==> default:  -- IP Forward:      
==> default:  -- External IP:     
==> default:  -- Autodelete Disk: true
```

* 實驗結束後，**別忘了關閉 Google Compute Engine 的虛擬機器喔**～以免試用的額度用完後，開始正式從信用卡扣款。

```
jazzwang: ~/vagrant-gcp/1_single_node_bigtop_ubuntu1404 $ vagrant destroy -f
==> default: Terminating the instance...
```

## Reference 參考資料

[1] Vagrant - https://www.vagrantup.com/
[2] vagrant-env - https://github.com/gosuri/vagrant-env
[3] vagrant-google - https://github.com/mitchellh/vagrant-google
[4] vagrant-hostmanager - https://github.com/smdahlen/vagrant-hostmanager
[5] Puppet - https://puppetlabs.com/
[6] Puppet on Ubuntu - https://help.ubuntu.com/14.04/serverguide/puppet.html
[7] 'puppet-cloudera' - Puppet Module to deploy Cloudera Manager and CDH - https://forge.puppetlabs.com/razorsedge/cloudera
[8] source of 'puppet-cloudera' - https://github.com/razorsedge/puppet-cloudera