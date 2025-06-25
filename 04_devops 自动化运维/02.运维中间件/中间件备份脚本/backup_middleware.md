```bash
#!/bin/bash

########
# backup mysql db 
# auther: cmzhu
########

## public parm
NOW_DATE_STR=$(date +"%Y%m%d%H%M%S")

# Static Parm
# mysql
MYSQL_USER=root
MYSQL_PASSWD=1234
MYSQL_HOST=172.17.0.5

# mongo
MONGO_USER=
MONGO_PASSWORD=
MONGO_HOST=

# volumes
BACKUP_BASE_PATH=/opt/mysql
MYSQL_BACKUP_DIR=${BACKUP_BASE_PATH}/mysql-backup
MONGO_BACKUP_DIR=${BACKUP_BASE_PATH}/mongo-backup
ETCD_BACKUP_DIR=${BACKUP_BASE_PATH}/etcd-backup


mkdir -p ${MONGO_BACKUP_DIR}
mkdir -p ${ETCD_BACKUP_DIR}

# extra db
MYSQL_EXTRA_DB=(
    performance_schema
    information_schema
    sys
)

## 公共函数
function job_success(){
    echo $(date +"%Y-%m-%d %H:%M:%S") -  $@
    exit 0
}

function job_failed(){
    echo $(date +"%Y-%m-%d %H:%M:%S") -  $@
    exit 1
}

function job_execution(){
    echo $(date +"%Y-%m-%d %H:%M:%S") -  $@
}

function usage() {
    echo "用法: $0 {mongo|mysql|etcd [arg]}"
    exit 1
}

function back_mysql(){
    MYSQL_CMD=(
        mysql 
        -u${MYSQL_USER} 
        -p${MYSQL_PASSWD} 
        -h${MYSQL_HOST} 
    )
    MYSQLDUMP_CMD=(
        mysqldump 
        -u${MYSQL_USER} 
        -p${MYSQL_PASSWD} 
        -h${MYSQL_HOST} 
    )

    # 检查 mysql 链接地址是否正常
    "${MYSQL_CMD[@]}" -e "SELECT 1" 2>/dev/null
    if [[ $? -ne 0 ]];then
        job_failed ${MYSQL_USER} connection error!! 
    fi
    mkdir -p ${MONGO_BACKUP_DIR}
    # 备份所有mysql 数据库
    all_mysql_backup_db=$("${MYSQL_CMD[@]}" -e "SHOW DATABASES;" 2>/dev/null | grep -vE "^Database$|$(IFS=\|; echo "${MYSQL_EXTRA_DB[*]}")")
    for db_name in ${all_mysql_backup_db[@]};do
        job_execution  start backup database ${db_name}
        "${MYSQLDUMP_CMD[@]}" $db_name | gzip > ${MYSQL_BACKUP_DIR}/mysql.${db_name}.${NOW_DATE_STR}.sql.gz
        if [[ $? -eq 0 ]]; then
            job_execution   ${db_name} backup success !!! 
        else 
            job_failed ${db_name} backup failed !!! 
        fi
    
    done

}

func backup_mongo(){
    echo TODO!!!
    mkdir -p ${MONGO_BACKUP_DIR}

}

func backup_etcd(){
    echo TODO!!!
}

function main(){

    case "$1" in 
        mongo)
            backup_mongo
            ;;
        back_mysql)
            back_mysql
            ;;
        *)
            usage
            ;;
    esac
}

main $@
```

