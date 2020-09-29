---
layout: post
title:  "Cosmos-SDK nameservice 24"
author: violetstair
categories: [ CosmosSDK ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: true
---

# Run REST routes

이제 CLI 쿼리 및 트랜잭션을 테스트한 기능을 REST 서버에서 동일하게 동작하는지 테스트해 보겠습니다.
이전에 실행했던 `nsd`를 그대로 두고 주소 수집 정보 조회부터 시작합니다

```bash
nscli keys show jack --address
nscli keys show alice --address
```

이제 다른 터미널 창에서 `rest-server`를 실행해 봅니다.

```bash
nscli rest-server --chain-id namechain --trust-node
```

그런 다음 다음과 같은 쿼리를 구성하고 실행할 수 있습니다.

> NOTE: 아래 제공된 예제에서 비밀번호화 buyer/owner 주소를 테스트 중인 데이터로 바꾸어 실행해야 합니다.

```bash
# jack의 시퀀스 및 계정 번호를 가져와서 아래 요청을 구성합니다
$ curl -s http://localhost:1317/auth/accounts/$(nscli keys show jack -a)
# > {"type":"auth/Account","value":{"address":"cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6","coins":[{"denom":"jackCoin","amount":"1000"},{"denom":"nametoken","amount":"1010"}],"public_key":{"type":"tendermint/PubKeySecp256k1","value":"A9YxyEbSWzLr+IdK/PuMUYmYToKYQ3P/pM8SI1Bxx3wu"},"account_number":"0","sequence":"1"}}

# alice의 시퀀스 및 계정 번호를 가져와 아래 요청을 구성합니다
$ curl -s http://localhost:1317/auth/accounts/$(nscli keys show alice -a)
# > {"type":"auth/Account","value":{"address":"cosmos1h7ztnf2zkf4558hdxv5kpemdrg3tf94hnpvgsl","coins":[{"denom":"aliceCoin","amount":"1000"},{"denom":"nametoken","amount":"980"}],"public_key":{"type":"tendermint/PubKeySecp256k1","value":"Avc7qwecLHz5qb1EKDuSTLJfVOjBQezk0KSPDNybLONJ"},"account_number":"1","sequence":"2"}}

# jack이 name을 구매하기 위한 raw transaction을 생성합니다
# NOTE: 이 요청은 특정 환경에 특화해야 합니다. 또한 "buyer"와 "from"은 동일한 address여야 합니다.
curl -XPOST -s http://localhost:1317/nameservice/names --data-binary '{"base_req":{"from":"'$(nscli keys show jack -a)'","chain_id":"namechain"},"name":"jack1.id","amount":"5nametoken","buyer":"'$(nscli keys show jack -a)'"}' > unsignedTx.json

# 그런다음 transaction을 사인합니다
# NOTE: 실제 환경에서는 raw transactrion은 클라이언트 측에서 서명되어야 합니다. 또한 alice의 계정에 대한 쿼리가 표시한 내용에 따라 순서를 조정해야 합니다.
nscli tx sign unsignedTx.json --from jack --offline --chain-id namechain --sequence 1 --account-number 0 > signedTx.json

# 마지막으로 사이닝한 트랜잭션을 브로드캐스팅 합니다
nscli tx broadcast signedTx.json
# > { "height": "266", "txhash": "C041AF0CE32FBAE5A4DD6545E4B1F2CB786879F75E2D62C79D690DAE163470BC", "logs": [  {   "msg_index": "0",   "success": true,   "log": ""  } ],"gas_wanted":"200000", "gas_used": "41510", "tags": [  {   "key": "action",   "value": "buy_name"  } ]}

# jack이 방금 구매한 name에 대한 값을 설정합니다
# NOTE: 특정 환경에 대해 이 요청을 특정화 해야 합니다. 또한 "owner"와 "from"은 동일한 address여야 합니다.
$ curl -XPUT -s http://localhost:1317/nameservice/names --data-binary '{"base_req":{"from":"'$(nscli keys show jack -a)'","chain_id":"namechain"},"name":"jack1.id","value":"8.8.4.4","owner":"'$(nscli keys show jack -a)'"}' > unsignedTx.json
# > {"check_tx":{"gasWanted":"200000","gasUsed":"1242"},"deliver_tx":{"log":"Msg 0: ","gasWanted":"200000","gasUsed":"1352","tags":[{"key":"YWN0aW9u","value":"c2V0X25hbWU="}]},"hash":"B4DF0105D57380D60524664A2E818428321A0DCA1B6B2F091FB3BEC54D68FAD7","height":"26"}

# 다시한번 사인된 데이터를 브로드캐스팅 합니다
nscli tx sign unsignedTx.json --from jack --offline --chain-id namechain --sequence 2 --account-number 0 > signedTx.json
nscli tx broadcast signedTx.json

# 방금 jack이 설정한 name의 값을 조회합니다
$ curl -s http://localhost:1317/nameservice/names/jack1.id
# 8.8.4.4

# 방금 jack이 구매한 name의 whois 정보를 쿼리 합니다
$ curl -s http://localhost:1317/nameservice/names/jack1.id/whois
# > {"value":"8.8.8.8","owner":"cosmos127qa40nmq56hu27ae263zvfk3ey0tkapwk0gq6","price":[{"denom":"STAKE","amount":"10"}]}

# Alice가 jack의 name을 구매합니다.
$ curl -XPOST -s http://localhost:1317/nameservice/names --data-binary '{"base_req":{"from":"'$(nscli keys show alice -a)'","chain_id":"namechain"},"name":"jack1.id","amount":"10nametoken","buyer":"'$(nscli keys show alice -a)'"}' > unsignedTx.json

# 다시한번 사인된 데이터를 브로드캐스팅 합니다
# NOTE: alice의 계정 쿼리에 따라 계정 번호가 1, 시퀀스 번호는 2로 변경되었습니다
nscli tx sign unsignedTx.json --from alice --offline --chain-id namechain --sequence 2 --account-number 1 > signedTx.json
nscli tx broadcast signedTx.json
# > { "height": "1515", "txhash": "C9DCC423E10E7E5E40A549057A4AA060DA6D6A885A394F6ED5C0E40AEE984A77", "logs": [  {   "msg_index": "0",   "success": true,   "log": ""  } ],"gas_wanted": "200000", "gas_used": "42375", "tags": [  {   "key": "action",   "value": "buy_name"  } ]}

# 이제 Alice는 더이상 jack에서 구입한 name이 필요하지 않으므로 삭제 합니다
# NOTE: 오직 owner만 name을 삭제할 수 있습니다. alice가 owner 이므로 jack에게서 구매한 name을 삭제할 수 있습니다.
$ curl -XDELETE -s http://localhost:1317/nameservice/names --data-binary '{"base_req":{"from":"'$(nscli keys show alice -a)'","chain_id":"namechain"},"name":"jack1.id","owner":"'$(nscli keys show alice -a)'"}' > unsignedTx.json

# 마지막으로 사인된 데이터를 브로드캐스팅 합니다
# NOTE: alice의 account 번호는 여전히 1이지만 시퀀스 번호는 3이 되었습니다.
nscli tx sign unsignedTx.json --from alice --offline --chain-id namechain --sequence 3 --account-number 1 > signedTx.json
nscli tx broadcast signedTx.json

# Alice가 방금 삭제한 name에 대한 whois 정보를 쿼리합니다
$ curl -s http://localhost:1317/nameservice/names/jack1.id/whois
# > {"value":"","owner":"","price":[{"denom":"STAKE","amount":"1"}]}
```

## Request Schemas

### `POST /nameservice/names` BuyName Request Body

```json
{
  "base_req": {
    "name": "string",
    "chain_id": "string",
    "gas": "string,not_req",
    "gas_adjustment": "string,not_req",
  },
  "name": "string",
  "amount": "string",
  "buyer": "string"
}
```

### `PUT /nameservice/names` SetName Request Body

```json
{
  "base_req": {
    "name": "string",
    "chain_id": "string",
    "gas": "string,not_req",
    "gas_adjustment": "strin,not_reqg"
  },
  "name": "string",
  "value": "string",
  "owner": "string"
}
```

### `DELETE /nameservice/names` DeleteName Request Body

```json
{
  "base_req": {
    "name": "string",
    "chain_id": "string",
    "gas": "string,not_req",
    "gas_adjustment": "strin,not_reqg"
  },
  "name": "string",
  "owner": "string"
}
```
