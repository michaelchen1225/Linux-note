# Linux-note

> 參考來源：[鳥哥的 Linux 私房菜](https://linux.vbird.org)

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
    