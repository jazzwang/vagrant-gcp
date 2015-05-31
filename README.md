# vagrant-gcp

這裡存放了一些關於如何使用 `vagrant-google` 擴充元件來佈署 Hadoop BigTop 與 Cloudera Manager 到 Google Compute Engine 的範例。

## Requirement

 * 註冊一個 Google Cloud Platform 的帳號
    - https://console.developers.google.com/billing/freetrial
    - 產生 Google Cloud Platform API 的 OAuth P12 金鑰
 * 註冊一個 Koding 的帳號
    - https://koding.com/Register

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

