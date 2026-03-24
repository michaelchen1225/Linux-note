# 快速清理沒用的檔案

> 最近遇到硬碟爆滿的問題，在不考慮 LVM 的情況下，想要快速清理掉一些沒用的檔案。

### 1. apt/yum 快取

APT (Debian/Ubuntu)

```bash
apt clean          # 清除所有已下載的 .deb 套件快取
apt autoclean      # 只清除舊版本的快取
apt autoremove     # 移除不再需要的相依套件
# 快取位置：/var/cache/apt/archives/
du -sh /var/cache/apt/archives/
```

* 清除 yum 快取：

```bash
sudo yum clean all
```

### 2. 系統日誌

```bash
# Systemd Journal 日誌（保留最近 7 天）
journalctl --vacuum-time=7d

# 或限制大小上限
journalctl --vacuum-size=500M

# 傳統 log 檔（/var/log/）
find /var/log -name "*.gz" -delete        # 壓縮的舊 log
find /var/log -name "*.1" -delete         # rotated log
> /var/log/syslog                          # 清空但保留檔案（小心）
```

### 3.Temp 暫存檔

```bash
rm -rf /tmp/*
rm -rf /var/tmp/*

# 注意：確認沒有服務正在使用這些目錄
lsof +D /tmp
lsof +D /var/tmp
```

### 4.舊的 Linux Kernel

```bash
# 查看目前使用的 kernel
uname -r

# APT 系統：移除舊 kernel
apt autoremove --purge

# YUM 系統
package-cleanup --oldkernels --count=1
```

### 5. 應用程式快取

```bash
# Docker（如果有的話，通常佔很大空間！）
docker system prune -a       # 移除所有未使用的 image、container、network
docker volume prune          # 移除未使用的 volume

# pip 快取
pip cache purge

# npm 快取
npm cache clean --force
```

### 6. Core Dump 檔案

```bash
bashfind / -name "core" -type f 2>/dev/null
find /var/crash -type f -delete
```

### 7. 使用者家目錄的垃圾

```bash
# 各種快取目錄
~/.cache/            # 瀏覽器、應用程式快取
~/.thumbnails/       # 縮圖快取

# 清除這些目錄下的檔案

rm -rf /home/*/.cache/*
rm -rf /home/*/.thumbnails/*
```

### 8. 將 log 的內容清空

有時候會有程式不斷寫入某個 log file，可使用 `truncate`清空內容但保留檔案，不會影響正在寫入的 process：

```bash
truncate -s 0 /path/to/logfile.log
```

### 8. 分析最大的垃圾檔案

參考：[disk-sapce](disk-space.md)

### 綜合腳本

```bash
#!/usr/bin/env bash
# =============================================================================
# disk_cleanup.sh - Professional Linux Disk Maintenance Tool
# =============================================================================

set -uo pipefail

# -- Global Variables ----------------------------------------------------------
DRY_RUN=false
OS_TYPE=""
PKG_MGR=""
DEFAULT_THRESHOLD=90

# -- Logging Functions ---------------------------------------------------------
log_info()  { echo -e "[\033[1;34mINFO\033[0m]  $*"; }
log_ok()    { echo -e "[\033[1;32mOK\033[0m]    $*"; }
log_warn()  { echo -e "[\033[1;33mWARN\033[0m]  $*"; }
log_err()   { echo -e "[\033[1;31mERROR\033[0m] $*" >&2; }
log_dry()   { echo -e "[\033[1;35mDRY\033[0m]   (Simulated) $*"; }

run() {
    if "$DRY_RUN"; then
        log_dry "$*"
    else
        eval "$@"
    fi
}

# -- OS Detection --------------------------------------------------------------
detect_os() {
    if [[ -f /etc/debian_version ]]; then
        OS_TYPE="debian"; PKG_MGR="apt"
    elif [[ -f /etc/redhat-release ]]; then
        OS_TYPE="redhat"
        command -v dnf &>/dev/null && PKG_MGR="dnf" || PKG_MGR="yum"
    else
        OS_TYPE="unknown"; PKG_MGR="unknown"
    fi
}

# -- Core Logic Functions ------------------------------------------------------

get_top_files() {
    local target=$1
    local count=$2
    echo -e "\n--- Top $count Largest Files (>1MB) in $target ---"
    echo "------------------------------------------------------------"
    find "$target" -xdev -type f -size +1M -exec ls -lh {} + 2>/dev/null \
        | sort -k5 -hr \
        | head -n "$count" \
        | awk '{printf "%-8s %s\n", $5, $9}'
    echo "------------------------------------------------------------"
}

run_disk_analysis() {
    echo -e "\n>>> Disk Usage Analysis"
    read -p "Directory to scan [/]: " target; target="${target:-/}"
    [[ ! -d "$target" ]] && { log_err "Directory not found."; return 1; }

    read -p "Alert threshold % [$DEFAULT_THRESHOLD]: " thresh; thresh="${thresh:-$DEFAULT_THRESHOLD}"

    local df_info=$(df -hP "$target" | tail -1)
    local usage=$(echo "$df_info" | awk '{print $(NF-1)}' | tr -d '%')
    
    log_info "Target: $target | Usage: ${usage}% | Threshold: ${thresh}%"

    if [[ "$usage" -ge "$thresh" ]]; then
        log_warn "Threshold reached! Listing potential issues:"
        echo -e "\n--- Top 10 Largest Directories ---"
        du -xh --max-depth=1 "$target" 2>/dev/null | sort -hr | head -n 11
        get_top_files "$target" 10
    else
        log_ok "Usage is normal. No detailed scan required."
    fi
}

find_top_files() {
    echo -e "\n>>> Top Files Explorer"
    read -p "Directory [/]: " target; target="${target:-/}"
    read -p "Number of files [10]: " count; count="${count:-10}"
    get_top_files "$target" "$count"
}

# -- Cleanup Tasks -------------------------------------------------------------

clean_temp_dirs() {
    log_info "Checking Temporary Directories (/tmp and /var/tmp)..."
    for dir in "/tmp" "/var/tmp"; do
        log_info "Analyzing $dir..."
        if [[ -n $(lsof +D "$dir" 2>/dev/null | grep -v "^COMMAND") ]]; then
            log_warn "$dir has files in use. Skipping to avoid system instability."
        else
            run "rm -rf ${dir:?}/*"
            log_ok "$dir has been cleared."
        fi
    done
}

clean_snap_old_versions() {
    log_info "Cleaning Disabled Snap Revisions (Keeping active)..."
    if command -v snap &>/dev/null; then
        snap list --all | awk '/disabled/{print $1, $3}' | while read snapname revision; do
            run "snap remove \"$snapname\" --revision=\"$revision\""
        done
        log_ok "Snap cleanup completed."
    fi
}

clean_pkg_cache() {
    log_info "Cleaning $PKG_MGR package cache & orphans..."
    if [[ "$PKG_MGR" == "apt" ]]; then
        run "apt-get clean"
        run "apt-get autoremove --purge -y"
    else
        run "$PKG_MGR clean all"
    fi
    log_ok "Package cleanup completed."
}

clean_journal() {
    log_info "Vacuuming Systemd Journal logs (older than 7d or >500MB)..."
    if command -v journalctl &>/dev/null; then
        run "journalctl --vacuum-time=7d --vacuum-size=500M"
    fi
}

# -- Menu ----------------------------------------------------------------------
show_menu() {
    echo -e "\n\033[1;36m[ ANALYSIS ]\033[0m"
    echo "  d) Disk usage Analysis (Usage % + Dirs + Files)"
    echo "  f) Find Top N Largest Files"
    echo -e "\033[1;36m[ CLEANUP  ]\033[0m"
    echo "  1) $PKG_MGR Cache & Orphans        2) Systemd Journal Logs"
    echo "  3) Temp Files (/tmp, /var/tmp)     4) Docker System Prune"
    [[ "$OS_TYPE" == "debian" ]] && echo "  5) Remove Old Snap Versions"
    echo -e "\033[1;36m[ ACTION   ]\033[0m"
    echo "  a) Run All Cleanups (1-4)          q) Quit"
    echo -ne "\n\033[1;33mAction >\033[0m "
}

main() {
    local args=()
    for arg in "$@"; do [[ "$arg" == "--dry-run" ]] && DRY_RUN=true || args+=("$arg"); done
    detect_os
    
    if [[ ${#args[@]} -gt 0 ]]; then
        for item in "${args[@]}"; do
            case "$item" in
                1) clean_pkg_cache ;; 2) clean_journal ;; 3) clean_temp_dirs ;; 
                4) run "docker system prune -f" ;; 5) clean_snap_old_versions ;;
                a) clean_pkg_cache; clean_journal; clean_temp_dirs; run "docker system prune -f" ;;
                d) run_disk_analysis ;; f) find_top_files ;;
            esac
        done
    else
        clear
        echo -e "\033[1;32mDisk Management Utility Initialized.\033[0m (Mode: $( "$DRY_RUN" && echo "DRY-RUN" || echo "LIVE" ))"
        while true; do
            show_menu
            read -r choice
            case "$choice" in
                q|Q) echo "Exiting."; exit 0 ;;
                d|D) run_disk_analysis ;;
                f|F) find_top_files ;;
                a|A) clean_pkg_cache; clean_journal; clean_temp_dirs; run "docker system prune -f" ;;
                1) clean_pkg_cache ;;
                2) clean_journal ;;
                3) clean_temp_dirs ;;
                4) run "docker system prune -f" ;;
                5) clean_snap_old_versions ;;
                *) log_err "Invalid selection: $choice" ;;
            esac
        done
    fi
}

main "$@"
```

