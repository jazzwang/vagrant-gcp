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
