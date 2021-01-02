---
layout: post
title:  "Keystore 파일 구조"
author: violetstair
categories: [ Programming ]
tags: [blockchain, programming, javascript]
image: assets/images/fawkes.jpg
description: "하나의 Privatekey로 여러 체인의 Address 생성해보기"
featured: false
hidden: false
---

## 블록체인의 서명과 키파일

블록체인에서는 공개키 암호화 방식을 이용해 사용자를 구분하고 사용자가 전송한 데이터가 유효한 데이터인지 검증하게 된다.
사용자는 자신의 `Private key`를 이용해 데이터를 서명을 하면 블록체인에서는 사용자의 `Public key`를 이용해 서면된 데이터를 검증하게 된다.
이때 사용되는 서명 알고리즘이 타원 곡선 알고리즘을 이용한 타원 곡선 디지털 서명 알고리즘(ECDSA)이다

### 타원 곡선 디지털 서명 알고리즘 (ECDSA : Elliptic Curve Digital Signature Algorithm)

타원 곡선 알고리즘을 이용한 디지털 서명 알고리즘

#### 타원 곡선 알고리즘

타원 곡선의 두 점을 지나는 직선의 값을 알더라도 곡선상의 대척점에 위치한 값을 알지는 못하는 이산 로그 문제 기반의 알고리즘
소인수분해 문제 기반의 `RSA`가 암호화 강도를 높이기 위해 키 길이를 늘릴 경우 계산 속도가 느려지는 문제가 있는 반면
타원 곡선 알고리즘의 경우 키 길이가 길어지더라도 속도는 영향이 없지만 암호화 강도는 기하 급수적으로 늘어나는 장점이 있다.

### Private key와 Address

1. `Private key`를 생성한다. 생성된 `Private key`는 랜덤함 값의 32 bytes의 길이를 가지는 Hex 데이터로 이루어진다

2. `Public key`는 `ECDSA`를 이용해 64 bytes의 공개키를 생성한다

3. `Public key`를 Hash 알고리즘인 `Keccak-256`으로 암호화한 Hash 데이터 32bytes 중 마지막 20 bytes에 `Address prefix` 값을 붙여 21 bytes의 주소를 생성할 수 있다
   `Address prefix`의 경우 `Ethereum`, `Klaytn`은 `0x`, `Icon`의 경우 `hx`를 사용한다.

### Private key를 암호화해 저장한 Keystore 파일

`Private key`를 보다 안전하게 보관 하기 위해 암호화된 내용으로 저장하게 되며 주요 블록체인의 `keystore` 파일의 내용은 다음과 같다.

#### Ethereum Keystore

```json
{
    "address": "aec00476b3eb165e7f27cfd7a6719415220807c6",
    "crypto": {
        "cipher": "aes-128-ctr",
        "cipherparams": {
            "iv": "4bdf21c5bab4dc0a46eaa23384685278"
        },
        "ciphertext": "6b51af57edcf74925584e6de93e4f61717a2075c0183b71ed103033830d04e74",
        "kdf": "scrypt",
        "kdfparams": {
            "dklen": 32,
            "n": 8192,
            "p": 1,
            "r": 8,
            "salt": "8bc74c04ef0abd806fa422591612809ff6493cba2cf43a3190c9065ea292b99a"
        },
        "mac": "352a248eaed1887699f8b357420f79943c939316c2653eadd7dcc0179896029b"
    },
    "id": "8bbcbe0e-0f62-4890-8545-b49dd13d9f0a",
    "version": 3
}
```

#### Klaytn Keystore

```json
{
    "address": "0xaec00476b3eb165e7f27cfd7a6719415220807c6",
    "id": "66e7acc5-f583-4544-9cd1-95a516141860",
    "keyring": [
        {
            "cipher": "aes-128-ctr",
            "cipherparams": {
                "iv": "3a54bd1cf3381997988bd5cf7334eec7"
            },
            "ciphertext": "4924d8cabf715e2778568efd1a1140fb62e855d345f6796294cf7ed10f4146f8",
            "kdf": "scrypt",
            "kdfparams": {
                "dklen": 32,
                "n": 4096,
                "p": 1,
                "r": 8,
                "salt": "d7203dc9276d18caea89d53ae393976b0020d7529dde0661bf6ccd63e2dad184"
            },
            "mac": "29e5332b8d90576a831dbb512b628f83807d487da670f961e665d0619f185700"
        }
    ],
    "version": 4
}
```

#### Icon Keystore

```json
{
    "address": "hx50f71d4aaa49f989db9214d5fc2c9d2d7e23f34c",
    "coinType": "icx",
    "crypto": {
        "cipher": "aes-128-ctr",
        "cipherparams": {
            "iv": "22c1e822e754d4bb0dd44b3bb509d250"
        },
        "ciphertext": "21b67001a9b0f0b73e7f0d8f9a2544c149519037045c4f21a599839ffeb02812",
        "kdf": "scrypt",
        "kdfparams": {
            "dklen": 32,
            "n": 16384,
            "p": 8,
            "r": 1,
            "salt": "c06d8cfc314ce2a3493a0feac599f729"
        },
        "mac": "e0dbbed9d99b4f520fb3e84794c3edcdab262aa7c5361ae4b72a8fc5ae32ab9f"
    },
    "id": "955b9951-eeb4-42bd-a12e-ed9da371b129",
    "version": 3
}
```

#### keystore 파일의 내용

* address : 블록체인에서 사용되는 주소
* coinType : 아이콘에만 있는 속성으로 체인에서 사용하는 Token의 symbol
* crypto : keystore 파일의 암호화 정보 (클레이튼은 keyring)
  * cipher : Private Key 암호화에 사용한 알고리즘의 이름
  * cipherparams : Private Key 암호화에 필요한 변수
    * iv : 암호화 벡터 값, 랜덤한 16 bytes
  * ciphertext : 위 알고리즘으로 Private Key를 암호화한 결과값
  * kdf (Key Derivation Function): 패스워드 암호화에 사용한 알고리즘의 이름
  * kdfparams : 패스워드 암호화에 필요한 변수
    * dklen(Derived Key Length) : 암호화 결과 값의 길이
    * n : 암호화 반복 횟수
    * p : 병렬화 수준
    * r : 기본 Hash Block Size
    * salt : 암호화에 필요한 salt 값 (32 bytes 랜덤 데이터)
  * mac : 암호문과 함께 만들어진 키의 마지막 16 bytes의 SHA3 Hash 값, keystore 파일 사용 시, 패스워드 입력값 검증을 위해 사용됨
* id : 랜덤한 값의 ID
* keyring : keystore 파일의 암호화 정보 (이더리움, 아이콘의 crypto에 해당)
* version : 버전 (이더리움, 아이콘 : 3, 클레이튼 : 4)

#### keystore 암호화

1. `keystore`의 `password`를 `kdf`에 정의된 블록 암호화를 이용해 Hash 값을(`Derived Key`) 얻어낸다
   `kdf`는 일반적으로 `scrypt`를 사용하지만 `pbkdf2`를 사용 하기도 한다.

2. 앞서 만들어진 `Derived Key`를 이용해 `Private key`를 `cipher`에 정의된 암호화 방식으(`aes-128-ctr`)로 암호화 한다
   이렇게 만들어진 데이터는 `cipertext`가 된다.
   즉 `Derived Key`가 `Private key`를 암호화하는 실제 password가 되는 것이다.

3. `Derived Key`의 마지막 16 bytes와 `cipertext`를 이어 붙인 48 bytes를 `SHA3-256`를 이용해 32 bytes의 Hash data를 만들며
   이 Hash data는 `mac` 이 된다.

#### keystore 복호화

1. 입력받은 `password`를 `kdf`에 정의된 블록 암호화를 이용해 Hash 값을(`Derived Key`) 얻어낸다.

2. 입력한 `password`가 올바른 값인지 확인하기위해 `cipertext`와 `Derived key`로 새로운 `mac`을 생성한 후 `keystore` 파일에 저장된 `mac`과 비교해 같을 경우 복호화 한다.

3. `Derived key`를 이용해 암호화된 `cipertext`를 복호화해 `Private key`를 얻어낸다.

## keystore 파일 만들기

```javascript
// 니모닉 코드 생성
const mnemonic = bip39.generateMnemonic(256, randomBytes, bip39.wordlists.korean)

// 니모닉 코드를 seed로 사용해 private key 생성
const seed = bip39.mnemonicToSeed(mnemonic)
const node = bip32.fromSeed(seed)
const child = node.derivePath("m/44'/60'/0'/0/0")
const ecpair = bitcoinjs.ECPair.fromPrivateKey(child.privateKey, {compressed : false})
const privateKey = ecpair.privateKey.toString('hex')

// 아이콘 wallet
const iconKeyWallet = IconService.IconWallet.loadPrivateKey(privateKey)
console.log('ICON Address : ', + iconKeyWallet.getAddress())
console.log('ICON Private Key : ' + iconKeyWallet.getPrivateKey())

// klaytn wallet
const klaytnKey = caver.wallet.keyring.createFromPrivateKey(privateKey)
console.log('Klaytn Address : ' + klaytnKey.address)
console.log('Klaytn Private Key : ' + klaytnKey.key.privateKey)

// ethereum account
const ethKey = accounts.privateKeyToAccount(privateKey)
console.log('Ethereum Address : ' + ethKey.address)
console.log('Ethereum Private Key : ' + ethKey.privateKey)

// cosmos-sdk account
console.log('Cosmos Address : ' + bech32.encode('cosmos', bech32.toWords(child.identifier)))
console.log('Cosmos Private Key : ' + privateKey)
```
