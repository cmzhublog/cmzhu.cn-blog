## nginx 升级版本脚本

用于漏洞修复:

[ CVE-2026-42945](https://avd.aliyun.com/detail/CVE-2026-42945?spm=5176.2020520154.sas.389.1e1aJrR7JrR75D&lang=zh)



```bash
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


function check_and_bak_nginx(){

    if command -v nginx &> /dev/null ;then
        job_execution "nginx 已安装"
        job_execution "nginx 版本为： $(nginx -v 2>&1 | grep -oE '[0-9]+\.[0-9]+\.[0-9]+' | head -n1)"
    
    else
        job_failed "nginx 未安装，请直接安装最新版本"
    fi

    if [ -d "/etc/nginx" ]; then
        cp -r /etc/nginx /etc/nginx.bak
    fi
    if [ -d "/usr/share/nginx/html" ]; then
        cp -r /usr/share/nginx/html /usr/share/nginx/html.bak
    fi
}

function upload_nginx(){

    # 获取 Nginx 版本号（如 1.30.0）
    nginx_ver=$(nginx -v 2>&1 | grep -oE '[0-9]+\.[0-9]+\.[0-9]+' | head -n1)

    if [ -z "$nginx_ver" ]; then
        job_failed "错误：未检测到 Nginx 或版本号"
    fi

    # 比较版本：将当前版本和 1.30 排序，若最小的那个是 1.30，则当前版本 >= 1.30
    if [ "$(printf '%s\n' "$nginx_ver" "1.30" | sort -V | head -n1)" = "1.30" ]; then
        job_execution "Nginx 版本 ($nginx_ver) >= 1.30"
    else
        job_execution "Nginx 版本 ($nginx_ver) < 1.30"
        wget https://nginx.org/packages/rhel/8/x86_64/RPMS/nginx-1.30.3-1.el8.ngx.x86_64.rpm  \
            && yum localinstall -y   nginx-1.30.3-1.el8.ngx.x86_64.rpm \
            && rm -f nginx-1.30.3-1.el8.ngx.x86_64.rpm
        wget https://dl.marmotte.net/rpms/redhat/el8/x86_64/nginx-1.30.1-1.el8.ex2/nginx-filesystem-1.30.1-1.el8.ex2.noarch.rpm \
            && yum localinstall -y nginx-filesystem-1.30.1-1.el8.ex2.noarch.rpm \
            && rm -f nginx-filesystem-1.30.1-1.el8.ex2.noarch.rpm
            
    fi
}

function main(){
    check_and_bak_nginx
    upload_nginx
    


}
main
```

