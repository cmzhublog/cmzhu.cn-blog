## TCP 抓包

```bash
#/bin/bash

set -e
# 定义常量
SCRIPT_NAME=$(basename "$0")
LOG_FILE="/var/log/${SCRIPT_NAME}.log"


PORTS=(
    "6443"
)


function job_success(){
    echo "$(date '+%Y-%m-%d %H:%M:%S') $*" >> "$LOG_FILE"
    exit 0 
}

function job_failed(){
    echo "$(date '+%Y-%m-%d %H:%M:%S') $*" >> "$LOG_FILE"
    exit 1
}

# 函数: 打印日志
function log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $*" >> "$LOG_FILE"
}


function tcpdump_ip(){
    
    for port in ${PORTS};do
        tcpdump_port_log="/var/log/${SCRIPT_NAME}_${port}.log"
        nohup  tcpdump -i any tcp port ${port}  >> "${tcpdump_port_log}"  2>&1 &
        sleep $(( RANDOM % 2 + 1 ))
        if [[ $? -ne 0 ]];then
            job_failed tcpdump 命令执行失败！！
        fi
    done
}


function  main(){
    tcpdump_ip
}

main

```

