# 建立 Parse 服務

## 目錄

* [概述說明](jian-li-parse-fu-wu.md#intro)
* [建立虛擬機](jian-li-parse-fu-wu.md#parse-vm)
* [申請與設定網域對應](jian-li-parse-fu-wu.md#parse-dns)
* [申請與設定 SSL 憑證](jian-li-parse-fu-wu.md#parse-ssl)
* [建立並設定 Parse Server 服務](jian-li-parse-fu-wu.md#parse-server)
* [建立並設定 Parse Dashboard 服務](jian-li-parse-fu-wu.md#parse-dashboard)
* [補充說明](jian-li-parse-fu-wu.md#supply)

## 概述說明 <a id="intro"></a>

Parse Service 在常見的配置上，包含**「資料庫服務」**用以存放資料，**「Parse 伺服器」**作為服務端口，**「Parse Dashboard」**提供簡單的管理之用。在這次的範例中，將會演示建立以 [Google Cloud Platform（下稱 GCP）](https://cloud.google.com/)的 Compute Engine「虛擬機」為基礎的方式，逐步架設 Parse 所需的**「Parse 伺服器」**、**「Parse Dashboard」**

> IaaS 服務可依需求、價格選用合適的服務，如：[AWS](https://aws.amazon.com/), [Linode](https://www.linode.com/)
>
> **大部分的 IaaS 服務商，通常都有提供免費試用計畫**，比如：[GCP 免費項目](https://cloud.google.com/free/), [AWS 免費方案](https://aws.amazon.com/tw/free/)

## 建立虛擬機 <a id="parse-vm"></a>

* 在 [GCP 的 Compute Engine](https://console.cloud.google.com/compute) 控制介面中，點擊「建立」，新增 VM Instance

![](../.gitbook/assets/parse-create.png)

* 設定 Instance 相關資訊
  * 名稱：自行命名
  * 區域：與資料庫同網域（因範例中資料庫伺服器網路預設開放 internal 連線）
  * 機器類型：依需求決定，範例選用微型機器
  * 開機磁碟：依需求決定，範例使用預設磁碟大小
  * 作業系統：範例選用 Ubuntu 14.04 LTS

![](../.gitbook/assets/parse-setup.png)

* 防火牆設定，開啟允許 HTTP & HTTPS

![](../.gitbook/assets/parse-firewall.png)

* 點開「管理、磁碟、網路、SSH 金鑰」，切換到「網路」頁籤，在「外部 IP」的選項中，點擊「建立 IP 位址」，填入名稱進行預約，讓此主機在未來即使重新啟動，都能綁定至此外部 IP

![](../.gitbook/assets/parse-external-ip.png)

* 等待建立完成後，便完成了一台虛擬機的啟用。爾後可以在此畫面，管理所有建立的虛擬機。也可透過後方的按鈕，直接透過 SSH 登入到每一台虛擬機

![](../.gitbook/assets/parse-instance-create.png)

## 申請與設定網域對應 <a id="parse-dns"></a>

由於安全性的考量，Parse 服務端口建議使用 SSL 連線，因此，必須先申請所想要使用的網域，並設定其 DNS Resource Record。市售的網域註冊服務很多，除了較為特別的網域名稱會被抬高價格之外，大部分的費用都差異不大（[按我查看 Google Domains 價格](https://support.google.com/domains/answer/6010092?hl=en)），建議可以挑選大型服務來購買，比如：[Google Domains](https://domains.google), [GoDaddy](https://godaddy.com)。以下將會演示，在 [noip](https://www.noip.com) 申請免費域名並設定 DNS 的過程

> noip 免費服務只能申請幾個限定網域的子網域，如果是要使用在正式產品，還是建議註冊一個漂亮的網域名稱吧

* 到[官方網站](https://www.noip.com/)，輸入網域並註冊帳號。範例中選了 ddns.net 網域，註冊了 parseServer 的子域名

![](../.gitbook/assets/no-ip-create.png)

* 完成帳號註冊與 Email 確認

![](../.gitbook/assets/confirm-no-ip-account.png)

* 到[設定畫面](https://my.noip.com/#!/dynamic-dns)，將網域的 IP 位址設定為剛剛建立的 Parse 虛擬機預約的外部 IP

![](../.gitbook/assets/my-no-ip-hostnames.png)

* 至此，您已經完成了網域以及其 DNS 設定。接下只需靜待 DNS Resource Record 更新，便可透過這個網域來連接設定好的 IP 位址。
* （選用）您可以透過本機端的終端機命令列來確認目前的 DNS 轉址，請記得將命令中的 DOMAIN\_NAME 替換成您申請的網址，比如：parseServer.ddns.net

  * Windows

  ```text
  ping DOMAIN_NAME
  ```

  * macOS

  ```text
  ping -c 4 DOMAIN_NAME
  ```

## 申請與設定 SSL 憑證 <a id="parse-ssl"></a>

接下來將會演示，如何申請免費的 [Let's Encrypt](https://letsencrypt.org/) SSL 憑證，與設定自動更新的過程。透過 SSH 登入 Parse 虛擬機，然後開始以下的步驟

* 安裝 mini-httpd，用來進行網域所有權驗證

  ```text
  sudo apt-get install mini-httpd -y
  ```

* 修改設定 /etc/default/mini-httpd

  * 編輯檔案，進入編輯模式

  ```text
  sudo nano /etc/default/mini-httpd
  ```

  * 將 START=0 修正成 START=1
  * 編輯完成後按下［control］+［x］離開，然後輸入［y］再鍵入［enter］確定寫入到原檔案

* 修改設定 /etc/mini-httpd.conf

  * 編輯檔案，進入編輯模式

  ```text
  sudo nano /etc/mini-httpd.conf
  ```

  * 將 host=localhost 修正成 host=0.0.0.0
  * 編輯完成後按下［control］+［x］離開，然後輸入［y］再鍵入［enter］確定寫入到原檔案

* 啟動 mini-httpd 服務

  ```text
  sudo /etc/init.d/mini-httpd start
  ```

* 此時可透過瀏覽器來測試 mini-httpd 是否正常運作

![](../.gitbook/assets/mini-httpd.png)

* 增加 Ubuntu 套件來源，過程中需按下［Enter］來同意繼續

  ```text
  sudo add-apt-repository ppa:certbot/certbot
  ```

* 安裝 Let's Encrypt 服務

  ```text
  sudo apt-get update
  sudo apt-get install letsencrypt -y
  ```

* 輸入以下命令，來驗證網域所有權，並取得 SSL 憑證

  > 請將 DOMAIN\_NAME 替換成您申請的網域名稱，EMAIL 替換成您的聯絡用 Email

  ```text
  sudo letsencrypt certonly -a webroot --agree-tos --webroot-path=/usr/share/mini-httpd/html -d DOMAIN_NAME -m EMAIL
  ```

* 驗證過程中，會詢問你是否願意分享 Email 給 [Electronic Frontier Foundation](https://zh.wikipedia.org/wiki/电子前哨基金会)，您可以輸入［Y/N］來［同意/不同意］。接著便會進入核發程序，成功的話，便會顯示 SSL 存放的位置。至此，您已經擁有您網域 SSL 所需要使用的憑證

  > Let's Encrypt 核發的憑證都只有 3 個月的有效期，將會在[補充說明](jian-li-parse-fu-wu.md#supply)中示範如何自動更新憑證

![](../.gitbook/assets/parse-ssl.png)

* 關閉並刪除 mini-httpd 服務

  ```text
  sudo /etc/init.d/mini-httpd stop
  sudo apt-get purge mini-httpd* -y
  ```

## 建立並設定 Parse Server 服務 <a id="parse-server"></a>

* 安裝 Node.js

  ```text
  curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
  sudo apt-get install nodejs -y
  ```

* 透過 npm 安裝 express-generator 的 package

  ```text
  sudo npm install express-generator -g
  ```

* 透過 express generator 建立基本的 express app 到 PARSE 資料夾，然後進到該資料夾中完成 npm 安裝

  > PARSE 可替換任意資料夾名稱

  ```text
  express PARSE
  cd PARSE
  sudo npm install
  ```

* 繼續在此資料夾中，透過 npm 安裝 parse-server 需要的 package

  ```text
  sudo npm install parse-server express
  ```

* 編寫 app.js 檔案，來完成 Parse Server 服務架設在 express 上

  * 先移除樣板 app.js 檔案

  ```text
  sudo rm app.js
  ```

  * 開始編輯 app.js，輸入以下指令進入編輯器

  ```text
  sudo nano app.js
  ```

  * 將以下 Parse Server 設定樣板貼入，將伺服器掛載在 express 之下

  ```text
  var ParseServer = require('parse-server').ParseServer;
  var api = new ParseServer({
  databaseURI: 'mongodb://PARSE_DB_USER:PARSE_DB_PASSWORD@0.0.0.0/PARSE_DB',
  appId: 'myAppId',        // 輸入您想要的的 App ID
  masterKey: 'myMasterKey' // 輸入您想要的 Master Key
  });

  var express = require('express');
  var app = express();
  app.use('/parse', api);

  var privateKey  = require('fs').readFileSync('/etc/letsencrypt/live/parseserver.ddns.net/privkey.pem', 'utf8');
  var certificate = require('fs').readFileSync('/etc/letsencrypt/live/parseserver.ddns.net/fullchain.pem', 'utf8');
  var credentials = {key: privateKey, cert: certificate};

  var httpsServer = require('https').createServer(credentials, app);
  httpsServer.listen(443);

  module.exports = app;
  ```

  > 將 databaseURI 中的 PARSE\_DB\_USER, PARSE\_DB\_PASSWORD, 0.0.0.0, PARSE\_DB，改成您 Parse 資料庫的「帳號」、「密碼」、「資料庫內部 IP」、「資料庫名稱」
  >
  > 將 privateKey, certificate 改成申請的憑證位置

  * 編輯結束後按下［control］+［x］離開，然後輸入［y］再鍵入［enter］確定寫入到 app.js

* 透過 npm 安裝常駐服務所需套件套件

  ```text
  sudo npm install forever forever-service -g
  ```

* 將此 express 封裝成 service

  > 請將 PARSE 改成您希望的 service 名稱

  ```text
  sudo forever-service install PARSE
  ```

* 至此，您已經完成了 Parse Server 的伺服器最基本設定。已經可以透過 各樣的 SDK 來連結您的 Parse Server 。此外，Parse 服務已經包裝成服務，您可以透過以下指令來簡單的控制 Parse 服務

  > 啟動服務 - "sudo service PARSE start"
  >
  > 停止服務 - "sudo service PARSE stop"
  >
  > 檢視服務狀態 - "sudo service PARSE status"
  >
  > 重新啟動服務- -"sudo service PARSE restart"

## 建立並設定 Parse Dashboard 服務 <a id="parse-dashboard"></a>

如果您有多個 Application Server，您可以透過以上的方法建立一台獨立的伺服器來運行 Parse Dashboard 服務。如果您只有建立一台 Application Server，您可以直接將 dashboard 建立在同一台伺服器中，甚至也可以直接掛載在 Parse Server 的 Express 之中

* 透過 npm 安裝 Parse Dashboard 所需套件套件

  ```text
  sudo npm install parse-dashboard -g
  ```

* 在 PARSE 資料夾中開始編輯 app.js，輸入以下指令進入編輯器

  ```text
  sudo nano app.js
  ```

  * 在原本掛載 Parse 服務的 Express 後，再掛載一個 Dashboard 到 /dashboard 端口

  ```text
  var express = require('express');
  var app = express();
  app.use('/parse', api); // 原本掛載的 Parse

  // Dashboard
  var ParseDashboard = require('parse-dashboard');
  var dashboard = new ParseDashboard({
   "apps": [ // Dashboard 中的 App 設定
    {
     "serverURL": "https://PARSE_SERVER/parse/",
     "appId": "myAppId",
     "masterKey": "myMasterKey",
     "appName": "AppNAME"
    }
   ],
   "users": [
     {
       "user":"", // 登入此 Dashboard 的帳號
       "pass":""  // 登入此 Dashboard 的密碼
     }
   ]
  });
  app.use('/dashboard', dashboard);
  ```

  * 編輯結束後按下［control］+［x］離開，然後輸入［y］再鍵入［enter］確定寫入到 app.js

* 重啟 PARSE 服務

```text
sudo service PARSE status
```

## 補充說明 <a id="supply"></a>

* 自動更新 Let's Encrypt SSL 憑證，運用伺服器 cron 系統，在每週日淩晨三點進行憑證更新以及重新啟動 Parse 服務

  > 請注意伺服器時區，預設是 Etc/UTC 時間

  ```text
  echo "0 3 * * 7 root /usr/bin/letsencrypt renew" | sudo tee -a /etc/crontab
  echo "0 3 * * 7 root service PARSE restart" | sudo tee -a /etc/crontab
  ```

