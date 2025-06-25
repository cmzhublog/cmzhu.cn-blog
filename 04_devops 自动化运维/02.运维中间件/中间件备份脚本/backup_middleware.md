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
MONGO_USER=work_rw
MONGO_PASSWORD=srjtRwkzs7qsUo
MONGO_HOST=10.52.3.160:27017,10.52.1.248:27017,10.52.1.196:27017
MONGO_DATABASE=(
    microloan
)

# volumes
BACKUP_BASE_PATH=/opt/backup-middleware
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
    echo 
    exit 0
}

function job_failed(){
    echo $(date +"%Y-%m-%d %H:%M:%S") -  $@
    echo 
    exit 1
}

function job_execution(){
    echo $(date +"%Y-%m-%d %H:%M:%S") -  $@
    echo 
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
            job_execution   ${db_name} backup success !!! 
        else 
            job_failed ${db_name} backup failed !!! 
        fi
    
    done

}


func backup_mongo(){
    ## check mongo login success 
    mongo \
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
    for mongo-database in "${MONGO_DATABASE[@]}" ; do
        job_execution  start backup mongo database ${db_name}
        if [ ! -d ${MONGO_BACKUP_DIR}/mongo.${mongo-database}.${NOW_DATE_STR} ]; then
            mkdir -p ${MONGO_BACKUP_DIR}/mongo.${mongo-database}.${NOW_DATE_STR}
        fi

        mongodump \
        --host ${MONGO_HOST} \
        -u ${MONGO_USER} \
        -p ${MONGO_PASSWORD} \
        --authenticationDatabase ${mongo-database} \
        --out ${MONGO_BACKUP_DIR}/mongo.${mongo-database}.${NOW_DATE_STR} \
        --gzip

        if [[ $? -eq 0 ]]; then
            job_execution   ${db_name} backup success !!! 
        else 
            job_failed ${db_name} backup failed !!! 
        fi
    done
    
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

