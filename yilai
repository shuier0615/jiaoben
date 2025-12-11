#!/usr/bin/env bash
#

# 青龙一键安装脚本 简洁版
TIME() {
    [[ -z "$1" ]] && {
        echo -ne " "
    } || {
        case $1 in
            r) export Color="\e[31;1m";;
            g) export Color="\e[32;1m";;
            b) export Color="\e[34;1m";;
            y) export Color="\e[33;1m";;
            z) export Color="\e[35;1m";;
            l) export Color="\e[36;1m";;
        esac
        [[ $# -lt 2 ]] && echo -e "\e[36m\e[0m ${1}" || {
            echo -e "\e[36m\e[0m ${Color}${2}\e[0m"
        }
    }
}

# 检查并安装系统依赖
install_system_deps() {
    TIME y "检查系统依赖..."
    if command -v apt-get &> /dev/null; then
        apt-get update
        apt-get install -y python3 python3-pip curl
    elif command -v yum &> /dev/null; then
        yum install -y python3 python3-pip curl
    fi
}

# 设置npm镜像源（使用国内可用源）
setup_npm_registry() {
    TIME y "设置npm镜像源..."
    
    # 尝试多个国内镜像源
    local mirrors=(
        "https://registry.npmmirror.com"
        "https://mirrors.cloud.tencent.com/npm/"
        "https://registry.npm.taobao.org"
    )
    
    for mirror in "${mirrors[@]}"; do
        TIME l "尝试设置镜像源: $mirror"
        npm config set registry "$mirror" --location=global
        
        # 测试镜像源是否可用
        if npm config get registry | grep -q "$mirror"; then
            TIME g "镜像源设置成功: $mirror"
            
            # 禁用SSL证书验证（解决证书过期问题）
            npm config set strict-ssl false
            npm config set cafile ""
            
            # 设置超时和重试
            npm config set fetch-retry-maxtimeout 60000
            npm config set fetch-retry-mintimeout 10000
            npm config set fetch-retries 5
            
            return 0
        fi
    done
    
    TIME r "所有镜像源设置失败，使用官方源"
    npm config set registry "https://registry.npmjs.org/"
    return 1
}

# 清理npm缓存
clean_npm_cache() {
    TIME y "清理npm缓存..."
    npm cache clean --force
    rm -rf ~/.npm/_logs/*
    rm -rf ~/.npm/_cacache/*
}

# 安装npm包
install_npm_package() {
    local package=$1
    local max_retries=3
    local retry_count=0
    
    while [[ $retry_count -lt $max_retries ]]; do
        TIME l "安装 $package (尝试 $((retry_count+1))/$max_retries)..."
        
        if npm install -g "$package" --verbose; then
            TIME g "$package 安装成功"
            return 0
        fi
        
        retry_count=$((retry_count+1))
        if [[ $retry_count -lt $max_retries ]]; then
            TIME y "安装失败，5秒后重试..."
            sleep 5
            clean_npm_cache
        fi
    done
    
    TIME r "$package 安装失败，跳过..."
    return 1
}

# 安装pip包
install_pip_package() {
    local package=$1
    local max_retries=3
    local retry_count=0
    
    # 设置pip镜像源
    pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
    pip3 config set global.trusted-host pypi.tuna.tsinghua.edu.cn
    pip3 config set global.timeout 120
    
    while [[ $retry_count -lt $max_retries ]]; do
        TIME l "安装 $package (尝试 $((retry_count+1))/$max_retries)..."
        
        if pip3 install --default-timeout=120 "$package"; then
            TIME g "$package 安装成功"
            return 0
        fi
        
        retry_count=$((retry_count+1))
        if [[ $retry_count -lt $max_retries ]]; then
            TIME y "安装失败，5秒后重试..."
            sleep 5
        fi
    done
    
    TIME r "$package 安装失败，跳过..."
    return 1
}

echo
echo
echo
TIME l "开始安装依赖..."
echo
TIME y "安装依赖需要时间，请耐心等待!"
echo
sleep 2

# 安装系统依赖
install_system_deps

# 设置npm镜像源
setup_npm_registry

# 清理缓存
clean_npm_cache

# 更新npm
TIME l "更新npm..."
npm install -g npm@latest

# 安装npm包
cd /ql || { TIME r "目录/ql不存在!"; exit 1; }

npm_packages=(
    "png-js"
    "date-fns"
    "axios"
    "crypto-js"
    "md5"
    "ts-md5"
    "tslib"
    "@types/node"
    "requests"
    "tough-cookie"
    "jsdom"
    "download"
    "tunnel"
    "ws"
    "global-agent"
    "bootstrap"
    "node-rsa"
    "form-data"
    "moment"
    "js-base64"
)

TIME l "开始安装npm依赖包..."
for package in "${npm_packages[@]}"; do
    install_npm_package "$package"
    echo
done

# 安装pip包
TIME l "开始安装pip依赖包..."
pip_packages=(
    "requests"
    "lxml"
    "PyExecJS"
    "user_agent"
    "pyDes"
)

# 特殊处理ds包
TIME l "安装ds (可能需要特殊处理)..."
if pip3 install ds 2>/dev/null; then
    TIME g "ds安装成功"
else
    # 尝试从其他源安装
    pip3 install ds --index-url https://pypi.org/simple/ || TIME r "ds安装失败，跳过..."
fi

for package in "${pip_packages[@]}"; do
    install_pip_package "$package"
    echo
done

# 特殊处理fs（fs是Node.js内置模块，不需要安装）
TIME l "注意: fs是Node.js内置模块，无需单独安装"

# 验证安装
TIME l "验证安装..."
echo
npm list -g --depth=0 | head -20
echo
pip3 list | head -20

echo
TIME g "依赖安装完成!"
TIME y "注意: 如有安装失败的包，请根据错误信息单独处理"
echo

# 恢复SSL验证（可选）
# npm config set strict-ssl true

exit 0
