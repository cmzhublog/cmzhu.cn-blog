## gitlab 对接ldap

参考文档： [https://docs.gitlab.com/ee/administration/auth/ldap/](https://docs.gitlab.com/ee/administration/auth/ldap/)

docker-compose.yml

```bash
version: '3'
services:

  gitlab-redis:
    restart: always
    image: docker.io/library/redis:6.2.13
    command: |
      redis-server --requirepass nsrItMx
    volumes:
      # - './redis-data:/data'
      - ${PWD}/gitlab/redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli","ping"]
      interval: 10s
      timeout: 5s
      retries: 40

  gitlab-postgresql:
    restart: always
    image: docker.io/library/postgres:13.11
    volumes:
      # - './pg-data:/var/lib/postgresql/data'
      - ${PWD}/gitlab/pg-data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: gitlab
      POSTGRES_PASSWORD: 5N1cnHnabc
      POSTGRES_DB: gitlab
    shm_size: '256m'
    healthcheck:
      test: ["CMD", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 40

  gitlab:
    image: 'docker.io/gitlab/gitlab-ee:16.3.4-ee.0'
    restart: always
    hostname: 'gitlab'
    dns:
      - 10.1.7.10
    depends_on:
      gitlab-redis:
        condition: service_healthy
      gitlab-postgresql:
        condition: service_healthy
    ports:
      - '10443:443'
      - '10080:80'
      - '11022:22'
    volumes:
      - './config:/etc/gitlab'
      - './logs:/var/log/gitlab'
      - './data:/var/opt/gitlab'
      - './certs:/certs'
      #- './gitlab/embedded/service/gitlab-rails/.license_encryption_key.pub:/opt/gitlab/embedded/service/gitlab-rails/.license_encryption_key.pub'
    shm_size: '256m'
    environment:
      TZ: 'Asia/Shanghai'
      GITLAB_TIMEZONE: Beijing
      GITLAB_ROOT_PASSWORD: YkOETuHabcd
      GITLAB_SKIP_UNMIGRATED_DATA_CHECK: 'true'
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://gitlab.cmzhu.cn'
        gitlab_rails['lfs_enabled'] = true
        gitlab_rails['gitlab_shell_ssh_port'] = 11022
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
        gitlab_rails['db_password'] = '5N1cnHnabc'
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
        #letsencrypt['enable'] = false

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
        gitlab_rails['gitlab_default_projects_features_builds'] = false

        # 关闭 Usage Statistics
        gitlab_rails['usage_ping_enabled'] = false
        gitlab_rails['sentry_enabled'] = false
        grafana['reporting_enabled'] = false

        # 关闭 GitLab Pages
        gitlab_pages['enable'] = false
        pages_nginx['enable'] = false

        # nginx
        nginx['client_max_body_size'] = '0'
        nginx['redirect_http_to_https'] = true
        nginx['ssl_certificate'] = "/certs/gitlab.cmzhu.cn.crt"
        nginx['ssl_certificate_key'] = "/certs/gitlab.cmzhu.cn.key"

        # ldap
        gitlab_rails['ldap_enabled'] = true
        gitlab_rails['ldap_servers'] = {
          'main' => {
            'label' => 'LDAP',
            'host' =>  '192.168.101.10',
            'port' => 389,
            'uid' => 'sAMAccountName',
            'bind_dn' => 'CN=Administrator,CN=Users,DC=cmzhu,DC=cn',
            'password' => 'abc123!',
            'encryption' => 'plain',
            'timeout' => 10,
            'active_directory' => true,
            'user_filter' => '',
            'base' => 'CN=Users,DC=cmzhu,DC=cn',
            'group_base' => 'CN=cmzhu,CN=Users,DC=cmzhu,DC=cn',
            'user_filter' => '(memberof=CN=cmzhu,CN=Users,DC=cmzhu,DC=cn)',
            'attributes' => {
              'username' => ['uid','userid','sAMAccountName'],
            },
            'lowercase_usernames' => 'false',
            'retry_empty_result_with_codes' => [80],
            'allow_username_or_email_login' => false,
            'block_auto_created_users' => false
          }
        }
```

