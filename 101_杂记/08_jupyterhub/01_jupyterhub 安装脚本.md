## jupyterhub 安装脚本

背景

- 支持一键安装jupyterhub 
- 支持用户创建

脚本install_jupyterhub.sh

```bash
#!/bin/bash

# JupyterHub 安装部署脚本
# 使用说明：以root用户执行此脚本

set -Eeuo pipefail  # 遇到错误、未定义变量或管道错误时立即退出

# 始终从脚本所在目录读取用户清单，避免因执行目录不同而找不到文件
SCRIPT_DIR="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)"

# 颜色定义
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# 日志函数
log_info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# 检查是否以root用户运行
check_root() {
    if [[ $EUID -ne 0 ]]; then
        log_error "此脚本必须以root用户运行"
        exit 1
    fi
    log_info "root权限检查通过"
}

# 安装系统依赖
install_dependencies() {
    log_info "安装系统依赖包..."

    # 优先根据包管理器判断，避免 /etc/os-release 导致 Debian 被误判为 RedHat
    if command -v dnf &>/dev/null; then
        dnf update -y
        dnf install -y python3 python3-devel python3-pip \
                      nodejs npm \
                      git curl wget \
                      sqlite sqlite-devel \
                      gcc gcc-c++ make
    elif command -v yum &>/dev/null; then
        yum update -y
        yum install -y python3 python3-devel python3-pip \
                      nodejs npm \
                      git curl wget \
                      sqlite sqlite-devel \
                      gcc gcc-c++ make
    elif command -v apt-get &>/dev/null; then
        apt-get update
        apt-get install -y python3 python3-dev python3-pip \
                          nodejs npm \
                          git curl wget \
                          sqlite3 libsqlite3-dev \
                          gcc g++ make

    else
        log_warn "未知的系统类型，请手动安装依赖"
    fi

    # 升级pip
    pip3 install --upgrade pip
    log_info "系统依赖安装完成"
}

# 安装JupyterHub和相关组件
install_jupyterhub() {
    log_info "安装JupyterHub和相关组件..."

    # 安装JupyterHub
    pip3 install jupyterhub

    # 安装Jupyter Notebook（用于单用户服务器）
    pip3 install notebook

    # 安装配置代理
    npm install -g configurable-http-proxy@3

    # 安装sudospawner（用于sudo创建用户）
    pip3 install sudospawner

    # 安装PAM认证（可选，用于系统用户认证）
    #pip3 install jupyterhub-pamauthenticator

    log_info "JupyterHub安装完成"
}

# 创建系统用户
create_users() {
    local users_file="${SCRIPT_DIR}/jupyterhub_users.txt"
    local username password uid extra
    local existing_uid home_dir primary_group
    local line_no=0
    local validation_errors=0
    declare -A seen_users=()
    declare -A seen_uids=()

    log_info "创建JupyterHub用户..."

    # 检查用户文件是否存在，如果不存在则创建示例
    if [ ! -f "$users_file" ]; then
        log_warn "用户文件 $users_file 不存在，创建示例文件"
        cat > "$users_file" << 'EOF'
# JupyterHub用户列表
# 每行格式：用户名:密码:UID（UID为必填项）
# 示例：
zhucongming:请修改此密码:1000
EOF
        chmod 600 "$users_file"
        log_info "示例用户文件已创建: $users_file"
        log_warn "请编辑此文件后重新运行用户创建步骤"
        return
    fi

    # 文件含明文密码，仅允许root读写
    chmod 600 "$users_file"

    # 第一遍只做校验，确保配置全部正确后再创建，避免产生部分用户
    while IFS=':' read -r username password uid extra || [[ -n "${username:-}" ]]; do
        ((line_no += 1))

        # 跳过空行和注释
        [[ "${username:-}" =~ ^[[:space:]]*# ]] && continue
        [[ -z "$username" ]] && continue

        if [[ -n "${extra:-}" ]]; then
            log_error "第 ${line_no} 行格式错误，应为：用户名:密码:UID"
            ((validation_errors += 1))
            continue
        fi

        if [[ ! "$username" =~ ^[a-zA-Z_][a-zA-Z0-9_.-]*[$]?$ ]]; then
            log_error "第 ${line_no} 行用户名不合法: $username"
            ((validation_errors += 1))
            continue
        fi

        if [[ -z "${uid:-}" ]]; then
            log_error "第 ${line_no} 行用户 $username 未指定 UID"
            ((validation_errors += 1))
            continue
        fi

        if [[ ! "$uid" =~ ^(0|[1-9][0-9]*)$ ]]; then
            log_error "第 ${line_no} 行用户 $username 的 UID 格式不合法: $uid"
            ((validation_errors += 1))
            continue
        fi

        if [[ -n "${seen_users[$username]:-}" ]]; then
            log_error "第 ${line_no} 行用户名重复: $username"
            ((validation_errors += 1))
            continue
        fi

        if [[ -n "${seen_uids[$uid]:-}" ]]; then
            log_error "第 ${line_no} 行 UID $uid 与用户 ${seen_uids[$uid]} 重复"
            ((validation_errors += 1))
            continue
        fi
        seen_users["$username"]=1
        seen_uids["$uid"]="$username"

        if id "$username" &>/dev/null; then
            existing_uid="$(id -u "$username")"
            if [[ "$existing_uid" != "$uid" ]]; then
                log_error "用户 $username 已存在，但现有 UID 为 $existing_uid，配置要求为 $uid"
                ((validation_errors += 1))
            fi
        elif getent passwd "$uid" &>/dev/null; then
            log_error "UID $uid 已被用户 $(getent passwd "$uid" | cut -d: -f1) 占用"
            ((validation_errors += 1))
        fi
    done < "$users_file"

    if ((validation_errors > 0)); then
        log_error "用户文件校验失败，共发现 ${validation_errors} 个错误；未创建任何用户"
        return 1
    fi

    # 第二遍创建用户；UID 必须通过 -u 显式传给 useradd
    while IFS=':' read -r username password uid extra || [[ -n "${username:-}" ]]; do
        [[ "${username:-}" =~ ^[[:space:]]*# ]] && continue
        [[ -z "${username:-}" ]] && continue

        if id "$username" &>/dev/null; then
            log_info "用户 $username 已存在，UID 已确认是 $uid"
        else
            useradd -m -u "$uid" -s /bin/bash "$username"

            # 设置密码
            if [ -n "$password" ]; then
                printf '%s:%s\n' "$username" "$password" | chpasswd
                log_info "创建用户: $username（UID: $uid）"
            else
                log_warn "已创建用户 $username（UID: $uid），但没有设置密码"
            fi

            # 创建Jupyter配置目录
            IFS=':' read -r _ _ _ _ _ home_dir _ < <(getent passwd "$username")
            primary_group="$(id -gn "$username")"
            mkdir -p "$home_dir/.jupyter"
            chown -R "$username:$primary_group" "$home_dir"
        fi
    done < "$users_file"

    log_info "用户创建完成"
}

# 配置JupyterHub
configure_jupyterhub() {
    log_info "配置JupyterHub..."

    # 创建配置目录
    mkdir -p /etc/jupyterhub
    cd /etc/jupyterhub

    # 生成默认配置
    if [ ! -f jupyterhub_config.py ]; then
        log_info "生成JupyterHub配置文件..."
        jupyterhub --generate-config

        # 备份原始配置
        cp jupyterhub_config.py jupyterhub_config.py.backup
    fi

    # 创建自定义配置
    cat > /etc/jupyterhub/custom_config.py << 'EOF'
# JupyterHub自定义配置
import os
from jupyterhub.spawner import LocalProcessSpawner

# 使用PAM认证
c.JupyterHub.authenticator_class = 'pam'
c.PAMAuthenticator.open_sessions = False

# 允许root运行hub
c.JupyterHub.allow_root = True

# 设置hub的IP和端口
c.JupyterHub.hub_ip = '0.0.0.0'
c.JupyterHub.hub_port = 8080

# 设置公共接口
c.JupyterHub.ip = '0.0.0.0'
c.JupyterHub.port = 8000

# 使用sudospawner
c.JupyterHub.spawner_class = 'sudospawner.SudoSpawner'

# 设置sudospawner路径
c.SudoSpawner.sudo_cmd = ['sudo', '-nHu', '{USERNAME}']

# 设置默认notebook服务器
c.Spawner.default_url = '/tree'
c.Spawner.notebook_dir = '~/'

# 用户白名单（可选）
# c.Authenticator.allowed_users = {'user1', 'user2'}
c.Authenticator.allow_all = True

# 管理员用户
c.Authenticator.admin_users = {'root'}

# SSL配置（如果需要HTTPS）
# c.JupyterHub.ssl_cert = '/path/to/cert.pem'
# c.JupyterHub.ssl_key = '/path/to/key.pem'

# Cookie密钥（生产环境需要设置）
import uuid
c.JupyterHub.cookie_secret = os.urandom(32)

# 数据库设置
c.JupyterHub.db_url = 'sqlite:////etc/jupyterhub/jupyterhub.sqlite'
EOF

    # 合并配置文件
    echo "" >> jupyterhub_config.py
    cat custom_config.py >> jupyterhub_config.py

    log_info "JupyterHub配置完成"
}

# 配置sudo权限
configure_sudo() {
    log_info "配置sudo权限..."

    # 创建sudospawner配置文件
    cat > /etc/sudoers.d/jupyterhub << 'EOF'
# Sudo配置 for JupyterHub
# 允许jupyterhub运行sudospawner

# 设置env_keep以保留环境变量
Defaults env_keep += "PATH"
Defaults env_keep += "PYTHONPATH"
Defaults env_keep += "HOME"
Defaults env_keep += "USER"

# 允许jupyterhub使用sudospawner
Cmnd_Alias JUPYTER_CMD = /usr/local/bin/sudospawner
%jupyterhub ALL=(ALL) NOPASSWD: JUPYTER_CMD
EOF

    # 设置正确的权限
    chmod 440 /etc/sudoers.d/jupyterhub

    log_info "sudo配置完成"
}

# 创建systemd服务
create_systemd_service() {
    log_info "创建systemd服务..."

    cat > /etc/systemd/system/jupyterhub.service << 'EOF'
[Unit]
Description=JupyterHub
After=network.target

[Service]
User=root
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
ExecStart=/usr/local/bin/jupyterhub -f /etc/jupyterhub/jupyterhub_config.py
WorkingDirectory=/etc/jupyterhub
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

    # 重新加载systemd
    systemctl daemon-reload
    systemctl enable jupyterhub

    log_info "systemd服务创建完成"
}

# 防火墙配置
configure_firewall() {
    log_info "配置防火墙..."

    # 检查防火墙类型
    if command -v ufw &> /dev/null; then
        # Ubuntu/Debian使用ufw
        ufw allow 8000/tcp
        ufw allow 8080/tcp
        log_info "UFW防火墙已配置"

    elif command -v firewall-cmd &> /dev/null; then
        # CentOS/RHEL使用firewalld
        firewall-cmd --permanent --add-port=8000/tcp
        firewall-cmd --permanent --add-port=8080/tcp
        firewall-cmd --reload
        log_info "Firewalld已配置"

    elif command -v iptables &> /dev/null; then
        # 使用iptables
        iptables -A INPUT -p tcp --dport 8000 -j ACCEPT
        iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
        log_info "iptables已配置"
    else
        log_warn "未找到支持的防火墙工具，请手动开放端口8000和8080"
    fi
}

# 显示安装完成信息
show_completion() {
    echo ""
    log_info "========================================="
    log_info "JupyterHub 安装完成！"
    log_info "========================================="
    echo ""
    log_info "访问地址: http://服务器IP:8000"
    log_info "Hub地址: http://服务器IP:8080"
    echo ""
    log_info "管理命令:"
    log_info "启动服务: systemctl start jupyterhub"
    log_info "停止服务: systemctl stop jupyterhub"
    log_info "查看状态: systemctl status jupyterhub"
    log_info "查看日志: journalctl -u jupyterhub -f"
    echo ""
    log_info "用户管理:"
    log_info "用户列表文件: ${SCRIPT_DIR}/jupyterhub_users.txt"
    log_info "用户格式: 用户名:密码:UID（UID必填且不可重复）"
    log_info "添加用户后重新运行脚本即可创建"
    echo ""
}

# 主函数
main() {
    if [[ "${1:-}" == "--create-users-only" ]]; then
        check_root
        create_users
        log_info "JupyterHub用户处理完成"
        return
    fi

    if [[ $# -gt 0 ]]; then
        log_error "未知参数: $1"
        echo "用法: $0 [--create-users-only]"
        exit 2
    fi

    log_info "开始安装JupyterHub..."

    # 执行安装步骤
    check_root
    install_dependencies
    install_jupyterhub
    create_users
    configure_jupyterhub
    configure_sudo
    create_systemd_service
#    configure_firewall

    log_info "启动JupyterHub服务..."
    systemctl start jupyterhub

    show_completion
}

# 脚本入口
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi

```



执行：

```bash
chmod +x install_jupyterhub.sh
./install_jupyterhub.sh
```



脚本可重复执行，执行第一次后不会创建普通用户，待编辑完jupyterhub_users.txt 文件后才会创建用户



