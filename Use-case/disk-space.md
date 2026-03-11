# 硬碟空間使用狀況

> 某個硬碟 or partition快滿了，如何找出哪個目錄/檔案占用最多空間？

## Step 1：列出硬碟用量

```bash
df -h
```

* `-h`：以人類可讀的格式顯示磁碟使用量（例如：GB、MB）。

從輸出可以看到每個 mount point 的使用率，再針對用量特別高的 partition 進行分析。


## Step 2：針對特定的 mount point 列出其中最大的目錄與檔案

```bash
du -xh --max-depth=1 <mount_point> 2>/dev/null | sort -hr | head -10
```

* `-x`：限制在同一個檔案系統內，不會跨越到其他 mount point。
* `-h`：以人類可讀的格式顯示大小。
* `--max-depth=N`：只列出第 N 層目錄的大小，這裡設定為 1 代表只列出第一層子目錄。


## Script

可以用以下腳本來輔助檢查硬碟使用情況：

**功能**：根據使用者提供的路徑與使用率閾值，檢查該路徑所在的 partition 的使用率，列出該路徑下最大的目錄，以及前10大的檔案。

```bash
#!/bin/bash

# --- Argument Logic ---
if [[ $# -eq 0 ]]; then
    TARGET_PATH="/"
    THRESHOLD=90
elif [[ $# -eq 1 ]]; then
    if [[ $1 =~ ^[0-9]+$ ]]; then
        TARGET_PATH="/"
        THRESHOLD=$1
    else
        TARGET_PATH=$1
        THRESHOLD=90
    fi
else
    TARGET_PATH=$1
    THRESHOLD=$2
fi

# Standardize path: Remove trailing slash
[[ "$TARGET_PATH" != "/" ]] && TARGET_PATH="${TARGET_PATH%/}"

# 1. Get Disk Info (Compatible with all versions of df)
# We read the 2nd line of df output
DF_INFO=$(df "$TARGET_PATH" | tail -1)

# 2. Extract Usage and actual Mount Point
# Use awk to get the 5th column (Usage%) and last column (Mount Point)
USAGE=$(echo "$DF_INFO" | awk '{print $(NF-1)}' | tr -d '%')
MOUNT_POINT=$(echo "$DF_INFO" | awk '{print $NF}')

if [ "$USAGE" -ge "$THRESHOLD" ]; then
    echo "WARNING: Filesystem [$MOUNT_POINT] (containing $TARGET_PATH) is ${USAGE}% full."
    echo "Threshold: ${THRESHOLD}%"
    echo

    echo "===== Largest directories under $TARGET_PATH ====="
    du -xh --max-depth=1 "$TARGET_PATH" 2>/dev/null | sort -hr | head -15
    echo

    echo "===== Top 10 largest files under $TARGET_PATH ====="
    find "$TARGET_PATH" -xdev -type f -size +1M -exec ls -lh {} + 2>/dev/null \
        | sort -k5 -hr \
        | head -10
else
    echo "Filesystem [$MOUNT_POINT] usage is ${USAGE}%, below threshold (${THRESHOLD}%)."
fi
```

* 使用方式：`./ck_disk.sh [target_path] [threshold]`
  * `target_path`：要檢查的目錄，預設為 `/`。
  * `threshold`：使用率的閾值，預設為 90%。

> 若 `target_path` 不是獨立的 partition，腳本會自動找出實際的 mount point 並檢查其使用率。不過還是會列出 `target_path` 下最大的目錄和檔案。

例如：

```bash
./ck_disk.sh /home 50
```
```text
WARNING: Filesystem [/] (containing /home) is 56% full. 
Threshold: 50%

===== Largest directories under /home =====
2.8G    /home
1.9G    /home/jice
892M    /home/centos
16K     /home/nagios

===== Top 10 largest files under /home =====
-rw------- 1 jice   jice   1005M Mar 11 15:49 /home/jice/workspace/realtime/nohup.out
-rw-rw-r-- 1 jice   jice    182M Jan 14  2019 /home/jice/application/jdk1.8.0_152.tar
(以下省略)
```

### 使用範例

* (預設) 檢查根目錄 `/` 的使用率，預設閾值為 90%：

```bash
./ck_disk.sh
```

* 檢查 `/home` 目錄的使用率，預設閾值為 90%：

```bash
./ck_disk.sh /home
```
> 若 /home 不是獨立的 partition，輸出訊息會提示實際的 mount point。

* 檢查根目錄 `/` 的使用率，閾值設定為 80%：

```bash
./ck_disk.sh 80
```

* 檢查 `/home` 目錄的使用率，閾值設定為 80%:

```bash
./ck_disk.sh /home 80
```

