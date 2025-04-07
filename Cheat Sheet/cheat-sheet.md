# Cheat sheet

## 目錄

## 基礎指令

### 熱鍵

* **Tab**：自動補齊檔案 or 指令
* **Ctrl + c**：中斷目前執行的指令
* **Ctrl + d**：結束 shell
* **Ctrl + l**：清除畫面
* **Ctrl + a**：游標移到行首
* **Ctrl + e**：游標移到行尾
* **Ctrl + u**：刪除游標前的字元
* **Ctrl + k**：刪除游標後的字元
* **Ctrl + r**：搜尋歷史指令
* **Alt + .**：重複上一個指令的最後一個 argument 
* **page up / page down**：上下翻之前執行的指令

### 相對路徑之表示

* `~`：當前使用者的 home 目錄
* `..`：上層目錄
* `.`：當前目錄

### pwd：顯示當前工作目錄

```bash
pwd
```

### cd：改變當前工作目錄

```bash
cd <path>
```

```bash
# 回到家目錄
cd 
```

```bash
# 回到上層目錄
cd ..
```

```bash
# 回到上上層目錄
cd ../..
```

```bash
# 回到上次的工作目錄
cd -
```

### ls：列出目錄/檔案的資訊

```bash
# 列出目錄下的檔案
ls <path>
```

```bash
# 列出目錄下的檔案(包含隱藏檔案)
ls -a <path>
```

```bash
# 列出目錄下的檔案並顯示詳細資訊
ls -l
```

```bash
# 讓 size 以人類可讀的方式顯示
ls -lh
```

```bash
# 依檔案大小排序
ls -lS
```

```bash
# 依檔案修改時間排序
ls -lt
```

```bash
# 查看目錄本身的詳細資訊
ls -ld <path>
```


## 權限設置

### chgrp：改變檔案所屬群組

```bash
chgrp <group> <file>
```
常用 option：

| Option | 說明 |
|--------|------|
| -R | 遞迴改變目錄下所有檔案的群組 |

### chown：改變檔案擁有者 or 群組

```bash
# 改 owner
chown <user> <file>
```

```bash
# 改 group
chown :<group> <file>
```

```bash
# 改 owner & group
chown <user>:<group> <file>
```

常用 option：

| Option | 說明 |
|--------|------|
| -R | 遞迴改變目錄下所有檔案的群組 |

### chmod：改變檔案權限

**數字法：r=4, w=2, x=1**

```bash
chmod 777 <file>
```

**字母法：r=read, w=write, x=execute**

> "+" 表示增加權限，"-" 號表示刪除權限，"=" 號表示設定權限

```bash
chmod u+x,g=rx,o=rx <file>
```

> 上面的 group & other 因位同樣都賦予 rx，可合併設定：

```bash
chmod u+x,go=rx <file>
```

常用 option：

| Option | 說明 |
|--------|------|
| -R | 遞迴改變目錄下所有檔案的群組 |