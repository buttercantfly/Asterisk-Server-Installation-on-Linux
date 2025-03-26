# Asterisk Server Installation on Linux

此 repository 旨在 Linux (Ubuntu) 上快速架設 SIP VoIP Server (Asterisk)，主要檔案皆為參數檔，詳細的安裝流程請參考 hackmd 文章：

https://hackmd.io/@buttercantfly/B1C9V-pIJx

架設完成後，即可透過手機上的網路電話軟體，如 Linphone 進行連接與網路通話，詳細設定請參考文章內容。


## Installation Guide
1. 更新系統
    ```bash!
    sudo apt update && sudo apt upgrade
    ```
2. 安裝編譯 Asterisk 和其模組所需的工具庫
    ```bash!
    sudo apt install -y build-essential wget git libncurses5-dev libssl-dev libxml2-dev libsqlite3-dev uuid-dev     libjansson-dev libiksemel-dev pkg-config libedit-dev
    # sudo apt install wget build-essential subversion
    ```
3. 下載 Asterisk 並解壓縮
    ```bash!
    cd /usr/src/
    sudo wget https://downloads.asterisk.org/pub/telephony/asterisk/asterisk-20-current.tar.gz
    sudo tar zxf asterisk-20-current.tar.gz
    cd asterisk-20.11.1/
    ```
4. 安裝編碼與聲音文件
    ```
    sudo contrib/scripts/get_mp3_source.sh
    sudo contrib/scripts/install_prereq install
    ```
5. 編譯與安裝 Asterisk
    ```bash!
    sudo ./configure
    sudo make menuselect
    sudo make -j6
    -------------------------
    sudo make install
    sudo make samples
    sudo make config
    sudo ldconfig
    ```
    
### Start Asterisk Server
```
sudo systemctl start asterisk
```

設定 server port 與 防火牆設定
```
sudo ufw allow 5060/udp
# sudo ufw enable (開啟時會阻擋未設定的 ip)
```

進入 server CLI 介面
```
sudo asterisk -rvv
```
![image](https://github.com/user-attachments/assets/69fbb6af-2c47-4328-8efa-519073b52447)


### Configuration File Settings
請依照 repository 中的檔案修改下圖中的三個檔案：
![image](https://github.com/user-attachments/assets/08f01767-ad3c-4f19-83f8-124e9dc5a004)
詳細的參數設定請自行閱讀 Default 中的原始參數檔案。

### Internet Phone Call Test

1. 下載任意網路通話程式，以 Linphone 為例：
2. 進入設定
3. 輸入 server IP（在網域/ Domain欄位），以及前面在 `pjsip.conf` 內所設定的帳號密碼
4. 輸入 `7001` 即可進行通話（需要有另一台手機作為另一個用戶端）


## Issues

### 確保 Windows 主機監聽 5060 端口 （當在windows上架設虛擬linux機器作為server時）
如果需要讓 Windows 主機代理 SIP 流量，**必須在主機上設置防火牆規則，允許外部設備通過 5060 端口進行通信**，同理**只要有防火牆開啟都需要去開啟對應的 port**。

#### 檢查防火牆規則
- 打開 控制台 → Windows Defender 防火牆 → 高級設定。
- 創建一條**輸入規則**：
    * 協議：UDP
    * 端口：5060
    * 動作：允許連接。
- 同樣，創建一條**輸出規則**。
    ![image](https://github.com/user-attachments/assets/b51f3939-e700-407d-998c-f20a479f87f9)

### 確保只有一個 Asterisk 實例在運行
執行以下命令檢查是否有多個 Asterisk 進程在運行，否則會導致資料庫更新異常：

```bash
ps aux | grep asterisk
```
如果有多個 Asterisk 進程，請終止多餘的進程：
```bash
sudo kill <PID>
```
