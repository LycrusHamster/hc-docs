# Huobi-chain 入门

## 安装和运行

### 安装依赖

#### MacOS

```
brew install autoconf libtool
```

#### ubuntu

```
apt update
apt install -y git curl openssl cmake pkg-config libssl-dev gcc build-essential clang libclang-dev
```

#### centos7

```
yum install -y centos-release-scl
yum install -y git make gcc-c++ openssl-devel llvm-toolset-7

# 打开 llvm 支持
scl enable llvm-toolset-7 bash
```

#### archlinux

```
pacman -Sy --noconfirm git gcc pkgconf clang make
```

### 直接下载预编译的二进制文件
 
我们会通过 [github releases](https://github.com/HuobiGroup/huobi-chain/releases) 发布一些常用操作系统的预编译二进制文件。如果其中包含你的操作系统，可以直接下载对应的文件。
 
### 从源码编译

#### 获取源码

通过 git 下载源码：

```
git clone https://github.com/HuobiGroup/huobi-chain.git
```

或者在 [github releases](https://github.com/HuobiGroup/huobi-chain/releases) 下载源码压缩包解压。

#### 安装 rust

参考： <https://www.rust-lang.org/tools/install>

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

#### 编译

```
cd /path/to/huobi-chain
make prod
```

编译完成后的二进制文件在 `target/release/huobi-chain`。

### 运行单节点

```
cd /path/to/huobi-chain

# 使用默认配置运行 huobi-chain
# 如果是直接下载的 binary，请自行替换下面的命令为对应的路径
./target/release/huobi-chain

# 查看帮助
$ ./target/release/huobi-chain  -h
Huobi-chain v0.1.0
Muta Dev <muta@nervos.org>

USAGE:
    huobi-chain [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -c, --config <FILE>     a required file for the configuration [default: ./devtools/chain/config.toml]
    -g, --genesis <FILE>    a required file for the genesis json [default: ./devtools/chain/genesis.json]
```

### 运行多节点

1. 根据节点拓扑，修改配置文件 config.toml，主要注意其中的 privkey、network 和 verifier_list 部分，可以参考下面的 docker-compose 配置，或者详细阅读下文的配置说明；
2. 将 huobi-chain binary 文件、huobi-chain 配置 config.toml 和创世块文件 genesis.json 分发到待部署的节点机器；
3. 启动 bootstrap 节点；
4. 启动其它节点；

## 配置说明

默认的配置样例在 `./devtools/chain/config.toml`，此处对其中的一些字段进行说明。

```toml
# chain id，链的唯一标识，同一个链的所有节点该项配置必须相同
chain_id = "b6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036"  # by sha256(Huobi-chain)

# 节点私钥，节点的唯一标识，在作为 bootstraps 节点时，需要给出地址和该私钥对应的公钥让其他节点连接；如果是出块节点，该私钥对应的地址需要在 consensus verifier_list 中
privkey = "45c56be699dca666191ad3446897e0f480da234da896270202514a0e1a587c3f"

# db config，链数据所在目录
data_path = "./devtools/chain/data"

[graphql]
# graphql 监听地址
listening_address = "0.0.0.0:8000"
# graphql 访问路径
graphql_uri = "/graphql"
# graphiql 路径
graphiql_uri = "/graphiql"

[network]
# p2p 监听地址
listening_address = "0.0.0.0:1337"

[[network.bootstraps]]
# 初始启动时访问的节点信息
pubkey = "031288a6788678c25952eba8693b2f278f66e2187004b64ac09416d07f83f96d5b"
address = "0.0.0.0:1888"

# 交易池相关配置
[mempool]
# 最大超时间隔，如果 当前区块数 + timeout_gap > tx 中的 timeout 字段，则交易池会拒绝接收该交易
timeout_gap = 20
# 交易池大小
pool_size = 20000
# 为了增加性能，每积累到这么多个交易才对外广播一次
broadcast_txs_size = 200
# 交易池广播交易间隔，单位为 毫秒(ms)
broadcast_txs_interval = 200

[consensus]
# 最大 cycles 限制
cycles_limit = 99999999
# cycle 价格
cycles_price = 1
# 出块间隔，单位为 毫秒(ms)
interval = 3000
# 出块节点的地址合集
verifier_list = [ "10f8389d774afdad8755ef8e629e5a154fddc6325a" ]

# 共识相关配置
[consensus.duration]
# 下面两项标识 propose 阶段的超时时间占共识间隔的比例的分子和分母。
# 按照上述配置为 3000ms，则 propose 共识阶段的超时时间为 3000ms * 24 / 30 = 2400ms。
# 下面类似的有 prevote 和 precommit 阶段的超时设置。
propose_numerator = 24
propose_denominator = 30
prevote_numerator = 6
prevote_denominator = 30
precommit_numerator = 6
precommit_denominator = 30

[executor]
# 设为 true 时，节点将只保存最新高度的 state
light = false
```