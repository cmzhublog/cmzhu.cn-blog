## gitlab 搭建方式



docker-compose.yaml

```yaml
version: '3'
services:

  gitlab-redis:
    restart: always
    image: dockerhub.cmzhu.cn:5000/docker.io/library/redis:6.2.13
    command: |
      redis-server --requirepass nsrItMx
    volumes:
      # - './redis-data:/data'
      - './gitlab/redis-data:/data'
    healthcheck:
      test: ["CMD", "redis-cli","ping"]
      interval: 10s
      timeout: 5s
      retries: 40

  gitlab-postgresql:
    restart: always
    image: dockerhub.cmzhu.cn:5000/docker.io/library/postgres:13.11
    volumes:
      # - './pg-data:/var/lib/postgresql/data'
      - './gitlab/pg-data:/var/lib/postgresql/data'
    environment:
      POSTGRES_USER: gitlab
      POSTGRES_PASSWORD: pDwaqrvCi8VCxN
      POSTGRES_DB: gitlab
    shm_size: '256m'
    healthcheck:
      test: ["CMD", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 40

  gitlab:
    image: 'dockerhub.cmzhu.cn:5000/docker.io/gitlab/gitlab-ee:16.3.4-ee.0'
    restart: always
    hostname: 'gitlab.cmzhu.cn'
    depends_on:
      gitlab-redis:
        condition: service_healthy
      gitlab-postgresql:
        condition: service_healthy
    ports:
      - '20443:443'
      - '20080:80'
      - '20022:22'
    volumes:
      - './config:/etc/gitlab'
      - './logs:/var/log/gitlab'
      - './data:/var/opt/gitlab'
      - './certs:/certs'
    shm_size: '256m'
    environment:
      # https_proxy: http://10.244.244.2:8899
      # http_proxy: http://10.244.244.2:8899
      # all_proxy: socks5://10.244.244.2:8899
      # no_proxy: 127.0.0.1,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,localhost,.local,.alpha-quant.com.cn
      TZ: 'Asia/Shanghai'
      GITLAB_TIMEZONE: Beijing
      GITLAB_ROOT_PASSWORD: pDwaqrvCi8VCxN
      GITLAB_HTTPS: false
      GITLAB_OMNIBUS_CONFIG: |
        # external_url 'https://gitlab.cmzhu.cn' # 此处的端口是什么， 默认对应http or https 协议启动的网络端口就是哪一个？
        gitlab_rails['lfs_enabled'] = true
        gitlab_rails['gitlab_shell_ssh_port'] = 20022
        gitlab_rails['time_zone'] = 'Asia/Shanghai'

        # 关闭所有监控
        prometheus['enable'] = false
        prometheus_monitoring['enable'] = false
        alertmanager['enable'] = false
        node_exporter['enable'] = false
        redis_exporter['enable'] = false
        postgres_exporter['enable'] = false
        pgbouncer_exporter['enable'] = false
        gitlab_exporter['enable'] = false
        grafana['enable'] = false
        sidekiq['metrics_enabled'] = false

        # postgresql
        postgresql['enable'] = false
        gitlab_rails['db_adapter'] = 'postgresql'
        gitlab_rails['db_encoding'] = 'utf8'
        gitlab_rails['db_database'] = "gitlab"
        gitlab_rails['db_host'] = 'gitlab-postgresql'
        gitlab_rails['db_port'] = 5432
        gitlab_rails['db_username'] = "gitlab"
        gitlab_rails['db_password'] = 'pDwaqrvCi8VCxN'
        gitlab_rails['auto_migrate'] = true

        # redis
        redis['enable'] = false
        gitlab_rails['redis_host'] = "gitlab-redis"
        gitlab_rails['redis_port'] = 6379
        gitlab_rails['redis_password'] = "nsrItMx"
        gitlab_rails['redis_database'] = 0

        # PUMA
        puma['worker_processes'] = 2
        puma['min_threads'] = 4
        puma['max_threads'] = 4

        # 降低后台守护进程并发数
        sidekiq['max_concurrency'] = 8
        sidekiq['min_concurrency'] = 4

        # nginx
        nginx['worker_processes'] = 4

        # Mattermost
        mattermost['enable'] = false
        mattermost_nginx['enable'] = false

        # Terraform
        gitlab_rails['terraform_state_enabled'] = false

        # 关闭电子邮件相关功能
        gitlab_rails['smtp_enable'] = false
        gitlab_rails['gitlab_email_enabled'] = false
        gitlab_rails['incoming_email_enabled'] = false

        # 关闭默认的 CI 功能
        gitlab_ci['gitlab_ci_all_broken_builds'] = false
        gitlab_ci['gitlab_ci_add_pusher'] = false

        # 关闭 Usage Statistics
        gitlab_rails['usage_ping_enabled'] = false
        gitlab_rails['sentry_enabled'] = false
        grafana['reporting_enabled'] = false

        # 关闭 GitLab Pages
        gitlab_pages['enable'] = false
        pages_nginx['enable'] = false

        # nginx
        nginx['client_max_body_size'] = '0'
        nginx['redirect_http_to_https'] = false
        #nginx['ssl_certificate'] = "/certs/tls.pem"
        #nginx['ssl_certificate_key'] = "/certs/tls.key"

        # OAuth2.0
        gitlab_rails['omniauth_enabled'] = false
        gitlab_rails['omniauth_allow_single_sign_on'] = ['openid_connect']
        gitlab_rails['omniauth_block_auto_created_users'] = false
        gitlab_rails['omniauth_providers'] = [
          {
            name: 'openid_connect',
            label: "Alpha Quant",
            args:  {
              name: "openid_connect",
              scope: ['openid','profile','email'],
              response_type: "code",
              discovery: true,
              issuer: 'https://auth.alpha-quant.com.cn',
              client_auth_method: "query",
              uid_field: "preferred_username",
              client_options: {
                identifier: "alpha-quant-app",
                secret: "ZXhhbXBsZS1hcHAtc2VjcmV0",
                redirect_uri: "https://gitlab.alpha-quant.com.cn/users/auth/openid_connect/callback"
              }
            }
          }
        ]
```

