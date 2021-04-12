---
layout: post
title:  "Cosmos-SDK nameservice 23"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# Build and run the app

## Building the `nameservice` application

`nameservice`가 완성 되었습니다. 이 버전을 빌드하기 위해서는 **Go 1.13.0** 이상의 버전이 필요합니다.

`go mod`를 사용해본적이 없다면 일부 환경 변수를 설정해야 합니다.

```bash
mkdir -p $HOME/go/bin
echo "export GOBIN=\$GOPATH/bin" >> ~/.bash_profile
echo "export PATH=\$PATH:\$GOBIN" >> ~/.bash_profile
source ~/.bash_profile
```

이제 애플리케이션을 설치하고 실행할 수 있습니다.

튜토리얼을 거치지 않은 경우 아래 저장소에서 코드를 받을 수 있습니다.

```bash
# Clone the source of the tutorial repository
git clone https://github.com/cosmos/sdk-tutorials.git
cd sdk-tutorials
cd nameservice
```

```bash
# Install the app into your $GOBIN
make install

# Now you should be able to run the following commands:
nsd help
nscli help
```

## Running the live network and using the commands

애플리케이션에 대한 구성 및 `genesis.json` 파일과 트랜잭션에 대한 계정을 초기화하려면 다음을 실행하여 시작할 수 있습니다.

> _*NOTE*_: 만약 이전에 바이너리를 실행한 적이 있다면 `nsd unsafe-reset-all`을 사용하거나 홈 폴더를 다음과 같은 명령어로 `rm -rf ~/.nscli ~/.nsd` 모두 삭제하여 처음부터 시작할 수 있습니다.

> _*NOTE*_: 만약 Ledger용 Cosmos 앱을 사용하려면 `nscli keys add jack` 명령어 뒤에 `--ledger` 옵션을 추가하면 됩니다. 그게 전부 입니다. 서명하려면 `jack`이 Ledger 키로 인식되며 장치를 요구하게 됩니다.

> _*NOTE*_: 프로젝트의 루트 디렉토리에 있는 `init.sh`파일에는 `rm -rf ~/.nscli ~/.nsd` 명령도 포함되어 있습니다. 터미널에서 `./init.sh`을 실행하여 초기화와 관련된 명령어를 한번에 실행할 수 있습니다.

```bash
# 설정 파일 및 genesis 파일 초기화
# moniker는 노드의 이름입니다.
nsd init <moniker> --chain-id namechain

# CLI 설정 추가, 이렇게 추가하면 해당 옵션은 나중에 다시 넣지 않아도 된다
nscli config chain-id namechain
nscli config output json
nscli config indent true
nscli config trust-node true

# 프로젝트 구성 디렉토리(기본값 : ~/.nsd)에는 암호화 되지 않은 "test" 키를 저장하는데 프로덕션에서는 이 "test" 키를 **절대** 사용해서는 안됩니다. keyring-backend으 ㅣ다른 옵션에 대한 자세한 내용은 https://docs.cosmos.network/master/interfaces/keyring.html 에서 확인 가능합니다.
nscli config keyring-backend test

# `Address` 출력을 복사하고 나중에 사용할 수 있도록 저장합니다
# [optional]Ledger Nano S를 사용하려면 끝에 "--ledger"를 추가합니다
nscli keys add jack

# `Address` 출력을 복사하고 나중에 사용할 수 있도록 저장합니다
nscli keys add alice

# genesis 파일에 token이 있는 두 개의 계정을 추가
nsd add-genesis-account $(nscli keys show jack -a) 1000nametoken,100000000stake
nsd add-genesis-account $(nscli keys show alice -a) 1000nametoken,100000000stake

# "nscli config" 명령은 "nscli" 명령에 대한 구성을 저장하지만 "nsd"에 대한 구성은 저장하지 않으므로 여기에서 플래그로 keyring-backend를 선언해야 합니다.
nsd gentx --name jack <or your key_name> --keyring-backend test
```

> Note: 기본적으로 최소 값을 설정하므로 금액을 지정할 필요가 없습니다

genTx를 통해 genesis 트랜잭션을 생성한 후에는 genesis 파일에 추가해야 합니다, 그렇게되면 nameservice 체인이 실행될 때 유효성 검사를 진행하게 됩니다.

`nsd collect-gentxs`

genesis 파일이 올바른지 확인하려면 다음과 같이 실행합니다.

`nsd validate-genesis`

이제 `nsd start`를 통해 `nsd`를 시작할 수 있습니다. 로그 스트리밍을 통해 블록이 생성되는 것을 확인할 수 있습니다. 이 작업에는 몇 초가 걸립니다.

첫 번째 노드를 성공적으로 실행 했습니다.

```bash
# 먼저 account에 자산이 있는지 확인해 봅니다
nscli query account $(nscli keys show jack -a)
nscli query account $(nscli keys show alice -a)

# genesis 파일에서 token을 사용하여 name을 구입합니다.
nscli tx nameservice buy-name jack.id 5nametoken --from jack

# 방금 구입한 name의 값을 설정합니다
nscli tx nameservice set-name jack.id 8.8.8.8 --from jack

# 등록한 name에 대한 resolve 쿼리를 시도합니다
nscli query nameservice resolve jack.id
# > 8.8.8.8

# 방금 등록한 name에 대한 whois 정보를 쿼리합니다
nscli query nameservice whois jack.id
# > {"value":"8.8.8.8","owner":"cosmos1l7k5tdt2qam0zecxrx78yuw447ga54dsmtpk2s","price":[{"denom":"nametoken","amount":"5"}]}

# Alice가 jack의 name을 구매합니다
nscli tx nameservice buy-name jack.id 10nametoken --from alice

# Alice가 방금 jack에게서 구입한 name을 삭제 합니다
nscli tx nameservice delete-name jack.id --from alice

# 방금 삭제한 name에 대해 whois 정보를 쿼리합니다
nscli query nameservice whois jack.id
# > {"value":"","owner":"","price":[{"denom":"nametoken","amount":"1"}]}
```

## Run second node on another machine (Optional)

nsd와 nscli를 설치하기 위해 방금 만든 명령을 실행하기위해 터미널을 엽니다

## init use another moniker and same namechain

```bash
nsd init <moniker-2> --chain-id namechain
```

## overwrite ~/.nsd/config/genesis.json with first node's genesis.json

## change persistent_peers

```bash
vim /.nsd/config/config.toml
persistent_peers = "id@first_node_ip:26656"
```

첫 번째 머신의 노드 ID를 찾으려면 해당 머신에서 다음 명령을 실행합니다.

```bash
nsd tendermint show-node-id
```

## start this second node

```bash
nsd start
```

### Cosmos SDK 애플리케이션을 구축했습니다. 튜토리얼을 완료 했습니다. REST 서버를 사용하여 동일한 명령을 실행하는 방법을 확인해 봅니다
