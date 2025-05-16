# Linux-note

> 參考來源：[鳥哥的 Linux 私房菜](https://linux.vbird.org)

> 舊筆記都是 PDF，放在[這個](https://drive.google.com/drive/folders/1piXt6WAo-I7DGQd1afMOIhf2zSKbH4c2?hl=zh-tw)雲端，最近會陸續整理至這個 repo

## 目錄

### [01 Linux 歷史](./01%20Linux%20歷史/01.Linux%20歷史.md)

* 簡介 Linux 的歷史，包括：何謂作業系統、什麼是 Linux、GNU 計畫等等。

### [02 Linux 初探](./02%20Linux%20初探/02.md)

* 包含初學文字介面的基本概念，包括：
  * 如何學習 Linux？
  * 終端介面
  * Linux 命令行 = Shell prompt + 指令
  * Linux 指令 = command + options + arguments
  * 常用熱鍵
  * 特殊符號
  * 萬用字元 *
  * 忘記指令怎麼辦？

### [03 檔案權限與目錄配置](./03%20檔案權限與目錄配置/03.md)

* 基本的權限概念，以及對 Linux 目錄樹架構的簡介：
  * 如何修改權限？chown、chgroup、chmod
  * rwx 對於檔案與目錄的意義
  * 常見操作至少需要的權限，例如：複製一個檔案最少需要的權限是什麼？
  * Linux 目錄樹架構
  * 絕對路徑 vs 相對路徑

### [04 Linux 檔案與目錄管理](./04%20Linux%20檔案與目錄管理/04.md)

* 許多常用且基本的指令，建議熟讀。依使用場景分類：
    * 目錄 & 路徑：$PATH、cd、pwd、mkdir、rmdir
    * 檔案與目錄管理：ls、cp、mv、rm、basename、dirname
    * 查看檔案內容：cat、more、less、head、tail、nl、od
    * 建立新檔 or 修改檔案時間：touch
    * 預設權限 & 隱藏權限：umask、SUID、SGID、sticky bit
    * 觀察檔案類型：file
    * 檔案的隱藏屬性：lsattr、chattr
    * 查檔案：whereis、which、find、locate、updatedb

### [05 Linux 磁碟與檔案系統管理](./05%20Linux%20磁碟與檔案系統管理/05.md)

  * 還沒寫完，只寫到 fdisk
  * 後續補上：鳥哥章節剩餘部分、/dev/mapper/centos-root 占滿的解法(LVM)

### [06 Linux 壓縮與打包](./06%20壓縮%20打包%20備份/06.md)

* 介紹 Linux 中的壓縮與打包工具，包含：
  * 壓縮：gzip、bzip2、xz
  * 打包：tar
  * 查閱壓縮檔內容：zcat、bzcat、xzcat.....

* 鳥哥的原文其實還包括 XFS 檔案系統的備份與還原、光碟燒錄等等用法，但其實這些都不常用到，且有些已經過時了，後續有空再補上。

### 補充：目前筆記的更新優先順序

我會選擇性的跳過一些段落或章節，這些跳過的部分大多數都和底層的硬體、原理有關，目前主要會優先更新的內容為：

* 上面已經更新過的內容

* Vim

* 認識與學習 bash shell

* 正規表達式 & 文件格式化處理

* 排程 

* 分析 log 檔

* 程序管理

* 系統服務 (service & daemon)

* 帳號管理 & ACL

### To-Do

* date 的用法 & date 於 shell script 中算時間的應用

* time 的用法 & time 於 shell script 中算時間的應用

* dd 用法 

* 將圖形介面關閉，轉為文字介面
