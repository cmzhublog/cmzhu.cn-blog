## docker-compose 实现平滑发版

### 实现逻辑

1、



```bash
SERVICE_BASE="mkdocs"
SERVICE_NAME="mkdocs"
SCALE_INSTANCES=2
HEALTH_CHECK_TIMEOUT=300

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
function rollout_upgrade(){
    container_all=($(docker ps -q -f "name=${SERVICE_BASE}-${SERVICE_NAME}-*"))
    for container in ${container_all}; do
        docker-compose stop ${container}
    done
    docker-compose up -d
    job_failed "❌ 新容器未达到健康状态，触发回滚"
}
# 获取当前服务的IP
OLD_CONTAINERS=($(docker ps -q -f "name=${SERVICE_BASE}-${SERVICE_NAME}-*" -f "health=healthy"))
if [[ ${#OLD_CONTAINERS[@]} -eq 0 ]]; then
    job_failed "❌  没有成功运行的容器，结束任务"
elif [[ ${#OLD_CONTAINERS[@]} -eq ${SCALE_INSTANCES} ]]; then
    rollout_upgrade
fi


function up_new_container(){
    docker-compose up -d --scale ${SERVICE_NAME}=${SCALE_INSTANCES} --no-recreate

    job_execution "⏳ 等待新容器就绪..."

    for i in $(seq 1 ${HEALTH_CHECK_TIMEOUT}); do
        NEW_CONTAINERS=($(docker ps -q -f "name=${SERVICE_BASE}-${SERVICE_NAME}-*" -f "health=healthy") )
        echo ${NEW_CONTAINERS}
        if [[ "${#NEW_CONTAINERS[@]}" -eq "${SCALE_INSTANCES}" ]]; then
           
            job_execution ${NEW_CONTAINERS}

            break
        fi
        sleep 1
    done

    if [ ${#NEW_CONTAINERS[@]} -lt ${SCALE_INSTANCES} ]; then
        rollout_upgrade
    fi

}


function update_nginx(){
    DOCKER_CMD=(
    "docker"
    "exec"
    "${SERVICE_BASE}-gateway-1"
    )

   "${DOCKER_CMD[@]}" nginx -t
   "${DOCKER_CMD[@]}" nginx -s reload
}

function offline_old_container(){
    echo "⏬ 逐步下线旧容器..."
    for container in ${OLD_CONTAINERS[@]}; do
        docker stop ${container} && docker rm ${container}
        sleep 5 # 间隔时间确保流量迁移
    done
    update_nginx
}


function main(){
    # up new container
    up_new_container
    # offline_old_container
    offline_old_container
}

main
```

