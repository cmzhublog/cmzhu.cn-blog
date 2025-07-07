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
MYSQL_USER=backup
MYSQL_PASSWD=1234
MYSQL_HOST=127.0.0.1

# mongo
MONGO_USER=work_rw
MONGO_PASSWORD=srjtRwkzs7qsUo
MONGO_HOST=10.52.0.1:27017,10.52.0.2:27017,10.52.0.3:27017
MONGO_DATABASE=(
    cmzhu
)

# etcd 
ETCD_ENDPOINTS="10.52.0.1:2379,10.52.0.2:2379,10.52.0.3:2379 "
ETCD_CERT_DIR="/opt/backup-data/cert/etcd"


# volumes
BACKUP_BASE_PATH=/opt/backup-data
MYSQL_BACKUP_DIR=${BACKUP_BASE_PATH}/mysql-backup
MONGO_BACKUP_DIR=${BACKUP_BASE_PATH}/mongo-backup
ETCD_BACKUP_DIR=${BACKUP_BASE_PATH}/etcd-backup



# extra db
MYSQL_EXTRA_DB=(
    performance_schema
    information_schema
    sys
)

## 公共函数
function job_success(){
    GREEN='\033[32m'
    NC='\033[0m'
    echo  -e "${GREEN} $(date +"%Y-%m-%d %H:%M:%S") -  $@ ${NC}"
    echo
    exit 0
}

function job_failed(){
    RED='\033[31m'
    NC='\033[0m'
    echo -e "${RED} $(date +"%Y-%m-%d %H:%M:%S") -  $@ ${NC}"
    echo
    exit 1
}

function job_execution(){
    echo -e  $(date +"%Y-%m-%d %H:%M:%S") -  $@
    echo
}

function usage() {
    echo "用法: $0 {backup_mongo|backup_mysql|backup_etcd [arg]}"
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
        job_failed ${MYSQL_USER} connection mysql server error!!
    fi


    if [ ! -d "${MYSQL_BACKUP_DIR}" ]; then
        mkdir -p ${MYSQL_BACKUP_DIR}
        job_execution create dir ${MYSQL_BACKUP_DIR} !!

    fi
    # 备份所有mysql 数据库
    all_mysql_backup_db=$("${MYSQL_CMD[@]}" -e "SHOW DATABASES;" 2>/dev/null | grep -vE "^Database$|$(IFS=\|; echo "${MYSQL_EXTRA_DB[*]}")")
    for db_name in ${all_mysql_backup_db[@]};do
        job_execution  start backup mysql database ${db_name}
        "${MYSQLDUMP_CMD[@]}" $db_name | gzip > ${MYSQL_BACKUP_DIR}/mysql.${db_name}.${NOW_DATE_STR}.sql.gz
        if [[ $? -eq 0 ]]; then
            job_execution   ${db_name} backup success ,backup file in ${MYSQL_BACKUP_DIR}/mysql.${db_name}.${NOW_DATE_STR}.sql.gz
        else
            job_failed ${db_name} backup failed !!!
        fi

    done

}


function backup_mongo(){
    ## check mongo login success
    mongosh \
    --username ${MONGO_USER}  \
    --password ${MONGO_PASSWORD} \
    --authenticationDatabase ${MONGO_DATABASE[0]} \
    --host ${MONGO_HOST} \
    --eval  "db.runCommand({connectionStatus:1})" \
    --quiet > /dev/null 2>&1

    if [[ $? -ne 0 ]]; then
        job_failed ${MONGO_USER} connection mongo server error!!
    fi

    ## check mongo backup dir already exist
    if [ ! -d "${MONGO_BACKUP_DIR}" ];then
        mkdir -p ${MONGO_BACKUP_DIR}
        job_execution create dir ${MONGO_BACKUP_DIR}
    fi

    ## backup mongo
    for mongo_database in "${MONGO_DATABASE[@]}" ; do
        job_execution  start backup mongo database ${db_name}
        if [ ! -d ${MONGO_BACKUP_DIR}/mongo.${mongo_database}.${NOW_DATE_STR} ]; then
            mkdir -p ${MONGO_BACKUP_DIR}/mongo.${mongo_database}.${NOW_DATE_STR}
        fi

        mongodump \
        --host ${MONGO_HOST} \
        -u ${MONGO_USER} \
        -p ${MONGO_PASSWORD} \
        --authenticationDatabase ${mongo_database} \
        --out ${MONGO_BACKUP_DIR}/mongo.${mongo_database}.${NOW_DATE_STR} \
        --gzip

        if [[ $? -eq 0 ]]; then
            job_execution   ${db_name} backup success ,backup file in ${MONGO_BACKUP_DIR}/mongo.${mongo_database}.${NOW_DATE_STR}
        else
            job_failed ${db_name} backup failed !!!
        fi
    done

}

function backup_etcd(){
    ETCD_CTL_CMD=(
        etcdctl 
        --endpoints="${ETCD_ENDPOINTS}"
        --cacert="${ETCD_CERT_DIR}/ca.crt" 
        --cert="${ETCD_CERT_DIR}/client.crt" 
        --key="${ETCD_CERT_DIR}/client.key"
    )
    # check ca file in the 
    if [ ! -f "${ETCD_CERT_DIR}/ca.crt" ] || [ ! -f  "${ETCD_CERT_DIR}/client.crt" ] || [ ! -f "${ETCD_CERT_DIR}/client.key" ];then
        job_failed etcd cert file not in ${ETCD_CERT_DIR}, please create etcd cert file \n \
        file name: \\n \
        ${ETCD_CERT_DIR}/ca.crt \\n \
        ${ETCD_CERT_DIR}/client.crt \\n \
        ${ETCD_CERT_DIR}/client.key
    fi
    ## check etcd backup dir already exist
    if [ ! -d "${ETCD_BACKUP_DIR}" ];then
        mkdir -p ${ETCD_BACKUP_DIR}
        job_execution create dir ${ETCD_BACKUP_DIR}
    fi
    export ETCDCTL_API=3 
    ${ETCD_CTL_CMD[@]} snapshot save ${ETCD_BACKUP_DIR}/etcd-${NOW_DATE_STR}.snapshot
    if [[ $? -eq 0 ]];then
        job_success etcd backup success,backup file ${ETCD_BACKUP_DIR}/etcd-${NOW_DATE_STR}.snapshot 
    else
        job_failed etcd backup failed !!! 
    fi
    ${ETCD_CTL_CMD[@]} snapshot status ${ETCD_BACKUP_DIR}/etcd-${NOW_DATE_STR}.snapshot

}

function main(){

    case "$1" in
        backup_mongo)
            backup_mongo
            ;;
        backup_mysql)
            backup_mysql
            ;;
        backup_etcd)
            backup_etcd
            ;;
        *)
            usage
            ;;
    esac
}

main $@
```

