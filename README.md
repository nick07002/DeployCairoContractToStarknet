# 部署 Cario1 的智能合约到 Starknet (测试网) 
Cairo [version 0.11](https://starkware.medium.com/starknet-alpha-v0-11-0-the-transition-to-cairo-1-0-begins-30442d494515) 版本释出后，可以直接部署 Cairo 1 到Starknet测试网/主网。 本文例子是测试网， 主网只需要改几个endpoint参数等即可
后面会再出教程





## 流程

- 克隆  Cairo 的repo, 需要制定指定的repo 的hash 高度
- 安装最新的  Cairo-lang, 用来和 Starknet交互。
- 用 cairo编译你的智能合约， 从 Cairo语言到 Sierra
- 用cairo-lang 声明Seirra合约
- 用cairo-lang 部署合约


## 克隆这个 repo
```bash
git clone https://github.com/nick07002/DeployCairoContractToStarknet
cd deploy-cairo1-demo
```

## 安装 Cairo 1 编译器

```bash
git clone https://github.com/starkware-libs/cairo/
cd cairo
git checkout 9c190561ce1e8323665857f1a77082925c817b4c
cargo build --all --release
```



到这里，你已经安装了 Cairo 的repo.

## 安装 Cairo-lang
回到当前repo的root位置
```bash
cd ..
```
设置python虚拟环境
```bash
python3.9 -m venv ~/cairo_venv_v11
source ~/cairo_venv_v11/bin/activate
```
如果你之前有安装过cairo-lang,需要卸载
```bash
pip3 uninstall cairo-lang
```
安装 Cairo lang
```bash
pip3 install ecdsa fastecdsa sympy
pip3 install cairo-lang
```
检查安装成功
```bash
starknet --version
```

## 用 Cairo 编译你的智能合约

进入cairo这个dir.
在此之前，如果你没有 `cargo` 需要安装 cargo, 这个是rust语言的包管理工具

```bash
curl https://sh.rustup.rs -sSf | sh

回复 1.
然后 source环境变量
source "$HOME/.cargo/env"

cd cairo
cargo run --bin starknet-compile -- ../hello_starknet.cairo ../hello_starknet.json --replace-ids	
```
祝贺，到这里你已经成功的编译了你的智能合约

## 用Cairo-lang声明合约
### 设置环境变量
```bash
export STARKNET_NETWORK=alpha-goerli
export STARKNET_WALLET=starkware.starknet.wallets.open_zeppelin.OpenZeppelinAccount
```

### 设置starknet的账户

我们这里安全起见，建立一个全新的测试网测试账户 
```bash
starknet new_account --account version_11
```
- 上面命令会返回一个账户地址
- 你需要用你已有的测试网的币打0.05eth左右到这个新账户。
- 从argent钱包发送后，点 Activity，关注这个tx的状态在"pening"以后就可以了。
```bash
starknet deploy_account --account version_11
```
上面会返回一个部署的TX的hash
关注这个部署协议的tx的状态，在"pening"以后就可以了。

### 声明合约
回到这个repo的root位置
```bash
cd ..
starknet declare --contract hello_starknet.json --account version_11
```
这时候你会收到了新的Class hash.这个就是你刚声明的合约地址。

### 问题及绝
如果你在declare这里出错，大概率你用的是mac的m1,m2芯片的电脑，这里有解决方案
```bash
Error: OSError: [Errno 8] Exec format error: '~/cairo_venv_11/lib/python3.9/site-packages/starkware/starknet/compiler/v1/bin/starknet-sierra-compile'
```
原因是cairo-lang用的rust是在其他架构下编译的，用下面的命令可以解决这个问题。

```bash
cp cairo/target/release/starknet-sierra-compile ~/cairo_venv_v11/lib/python3.9/site-packages/starkware/starknet/compiler/v1/bin/starknet-sierra-compile
```

再重新run上面的declare命令，应该就成功了

## 用Cairo-lang部署合约
 这里用到了上面的class hash.
```bash
starknet deploy --class_hash <class_hash> --account version_11
```
用starkscan (https://testnet.starkscan.co/) 检测你这个交易的状态，在L2 accepted之后就部署成功了

