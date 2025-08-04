## docker-compose 实现平滑发版

```bash
# 配置参数
SERVICE_NAME="web"          # 服务名称
NEW_IMAGE="your-app:latest" # 新镜像标签
SCALE_INSTANCES=3           # 目标实例数
HEALTH_CHECK_TIMEOUT=60     # 健康检查超时（秒）
NGINX_CONF="/etc/nginx/conf.d/app.conf" # Nginx配置路径

# 获取当前运行的容器ID
OLD_CONTAINERS=($(docker ps -q -f "name=${SERVICE_NAME}"))

# 启动新版本容器（数量与旧实例相同）
echo "▶️ 启动新版本容器..."
docker-compose up -d --scale ${SERVICE_NAME}=${SCALE_INSTANCES} --no-recreate

# 等待新容器健康状态
echo "⏳ 等待新容器就绪..."
for i in $(seq 1 ${HEALTH_CHECK_TIMEOUT}); do
  NEW_CONTAINERS=($(docker ps -q -f "name=${SERVICE_NAME}" -f "health=healthy"))
  if [ ${#NEW_CONTAINERS[@]} -ge ${SCALE_INSTANCES} ]; then
    break
  fi
  sleep 1
done

# 验证健康状态
if [ ${#NEW_CONTAINERS[@]} -lt ${SCALE_INSTANCES} ]; then
  echo "❌ 新容器未达到健康状态，触发回滚"
  docker-compose stop ${SERVICE_NAME} && docker-compose up -d
  exit 1
fi

# 更新Nginx配置（示例）
echo "🔄 更新负载均衡配置..."
cat > ${NGINX_CONF} <<EOF
upstream app_servers {
  ${NEW_CONTAINERS[@]/#/server } # 动态生成server列表
}
EOF
nginx -s reload

# 逐步停止旧容器
echo "⏬ 逐步下线旧容器..."
for container in ${OLD_CONTAINERS[@]}; do
  docker stop ${container} && docker rm ${container}
  sleep 5 # 间隔时间确保流量迁移
done

echo "✅ 滚动更新完成"
```

