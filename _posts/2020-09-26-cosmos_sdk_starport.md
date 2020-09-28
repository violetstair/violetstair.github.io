---
layout: post
title:  "Cosmos-SDK Starport"
author: violetstair
categories: [ CosmosSDK ]
tags: [Golang, Programming]
image: assets/images/starport.png
description: "COSMOS SDK Starport로 쉽게 만들어보는 vote app"
featured: false
hidden: true
---


![Application screenshot](/assets/images/starport/1.png)

사용자가 로그인, 투표 생성, 투표하기, 투표 결과 확인의 기능을 가지는 투표 어플리케이션을 만듭니다.
로그인한 사용자만 기능을 이용할 수 있으며, 투표 생성에는 토큰 200개가 필요지만, 투표는 무료입니다.

이번 작업에는 블록체인 빌드 도구인 [Starport](https://github.com/tendermint/starport)를 이용합니다. ([Starport 설치 방법](https://github.com/tendermint/starport#installation))
프로젝트 생성을 위해 다음과 같은 명령어를 실행합니다.

```bash
starport app github.com/alice/voter
```

Starport에 의해 `voter` 라는 디렉토리가 생성 되었으며, 기본적인 어플리케이션 구조를 생성한 것을 볼 수 있습니다.

`voter` 디렉토리 내용:

- `app` 에는 애플리케이션의 모든 기능을 연결하는 파일이 있습니다.
- `cmd`에는 사용자 어플리케이션과 상호작용하는 `voterd`와 `votercli` 프로그램이 위치합니다.
- `frontend`에는 위 스크린 샷에 보이는 것같은 사용자 웹 인터페이스가 위치합니다.
- `x` 에는 블록체인의 기능을 구성하는 모듈들이 위치합니다. 현재는 `voter`라는 모듈만 존재합니다.

프로젝트 디렉토리에 블록체인 기반 앱을 빌드하고 실행하는데 필요한 모든 코드가 준비되어 있습니다.
이제 starport를 이용해 앱을 실행할 수 있습니다.

```bash
starport serve
```

```markdown
📦 Installing dependencies...
🚧 Building the application...
💫 Initializing the chain...
🙂 Created an account. Password (mnemonic): fruit pudding eagle adapt keep junior loop brass trip place poverty gun goddess vehicle clap firm daughter bike analyst essay execute lesson shoe garlic
🙂 Created an account. Password (mnemonic): employ health rich wagon weather picture assist blade trade drill gas ankle loan tuition voyage virus upset news cross absent decide pizza achieve insect
🌍 Running a Cosmos 'voter' app with Tendermint.

🚀 Get started: http://localhost:12345/
```

단지 두 개의 명령만으로 블록체인 어플리케이션을 실행 했습니다. 이제 원하는 기능을 추가해볼 차례입니다.
투표 어플리케이션에는 `poll`, `voter` 두가지 타입이 존재합니다. `poll`은 `title`과 `options` 리스트를 포함 합니다.

## Adding polls

다음과 같은 명령어를 실행해 `poll` 아이템을 생성합니다.

```bash
starport type poll title options
```

이제 `starport serve`를 실행하고 [http://localhost:8080](http://localhost:8080)을 방문하면 `poll`이 생성된 것을 볼 수 있습니다.
어플리케이션을 다시 빌드하는데 약간의 시간이 걸릴 수 있으므로 몇 초 정도 기다려야 합니다.

![Application screenshot](/assets/images/starport/2.png)

콘솔에 출력된 비밀번호 중 하나로 로그인을 하고 `poll`을 생성해 봅니다.

입력 폼 위에 생성한 `poll`이 표시되었다면 `poll`을 블록체인에 성곡적으로 저장한 것 입니다.

아직까지 원하는 기능이 정상적으로 동작하진 않는것을 볼 수 있습니다.

`options` 필드를 추가(데이터를 배열로 저장)하고, 확인할 수 있는 버튼이 표시되어야 합니다.

그전에 `starport type` 명령으로 수정된 내용을 확인해 보겠습니다.

### `x/voter/types/TypePoll.go`

생성한 `Poll`의 타입을 정의합니다
`poll`에는 자동으로 생성된 두 개의 필드(creator, ID)와 사용자가 정의한 두 개의 필드(title, options)가 정의된 것을 확인할 수 있습니다.
`Options`는 문자열 리스트 형태로 저장하길 원하므로 **`string`을  `[]string`와 같은 형태로 수정합니다**

### `x/voter/types/MsgCreatePoll.go`

생성한 `poll`의 메시지 형태를 정의합니다

블록체인에 `title`과 `options`를 쓰거나 상태를 변경하기 위해 클라이언트는 HTTP POST 요청을 [http://localhost:1317/voter/poll](http://localhost:1317/voter/poll)에 전달해 변경합니다.
엔드포인트 핸들러는 `x/voter/client/rest/txPoll.go`에 정의 되어있습니다.

핸들러는 메시지 배열을 포함하는 서명되지않은 트랜잭션을 생성합니다. 그런 다음 클라이언트는 트랜잭션에 서명하고 [http://localhost:1317/txs](http://localhost:1317/txs)로 전달하게 됩니다.

그런 다음 애플리케이션은 각 메시지를 해당 핸들러 (여기서는 `x/voter/handlerMessageCreatePoll.go`)로 전송하여 트랜잭션을 처리합니다.

그런 다음 핸들러는 `x/voter/keeper/poll.go`에 정의 된 `CreatePoll` 함수를 호출하여 `poll` 데이터를 store에 기록합니다.
A handler then calls a `CreatePoll` function defined in `x/voter/keeper/poll.go` which writes the poll data into the store.

`MsgCreatePoll.go` 로 돌아가서 문자열 대신 목록으로 저장할 옵션을 만들어야합니다.

`MsgCreatePoll` 구조체에서 `Options string`을 `Options []string`으로 바꾸고 `NewMsgCreatePoll` 함수의 인수에서 `options string`을 `options []string`으로 바꿉니다.

### `x/voter/client/rest/txPoll.go`

`createPollRequest` 구조체에서 `Options string`을 `Options []string`로 바꿉니다.

### `x/voter/client/cli/txPoll.go`

사용자는 CLI를 통해 애플리케이션과 상호 작용할 수도 있습니다.

```bash
votercli tx voter create-poll "Text editors" "Emacs" "Vim" --from user1
```

이 명령은 `user1`의 개인 키를 이용하여 "create poll" 트랜잭션을 생성, 서명하고 블록체인에 전달하는 명령어이다.

여기서 `options`를 리스트로 받을 수 있도록 `argsOptions := string(args[1])`를 `argsOptions : = args [1 : len (args)]`와 같이 수정합니다.

이는 첫 번째 인수 이후의 모든 인수가 옵션의 목록을 나타낸다고 가정합니다.

이제 블록체인에에 필요한 모든 사항을 변경 했으므로 클라이언트 측 애플리케이션을 살펴 보겠습니다.

```bash
starport serve
```

### Front-end application

Starport는 `vote` 서비스에 대한 기본 프런트엔드 어플리케이션을 생성했습니다.
Starport has generated a basic front-end for our application.

프런트엔드는 [Vue.js](https://vuejs.org)를 프레임워크 상태 관리를 위해 [Vuex](https://vuex.vuejs.org/)와 함께 만들어 졌지만 모든 기능들은 HTTP API로 호출할 수 있으며 다른 언어의 프레임워크로도 개발할 수 있습니다.

대부분의 작업은 앱의 페이지 템플릿이 포함 된 `frontend/src/views` 디렉토리를 살펴볼 것입니다.
`frontend/src/store/index.js`는 `frontend/src/components`에 포함된 버튼 및 입력 폼과 같은 컴포넌트들을 이용해 블록체인에 트랜잭션 데이터를 주고 받습니다.

`frontend/src/store/index.js` 내에서 지갑 처리, 트랜잭션 생성, 서명 및 브로드 캐스팅을 위한 라이브러리 인 [CosmJS](https://github.com/cosmwasm/cosmjs)를 가져오고 Vuex 스토어를 정의합니다.

블록체인에 데이터를 보내기 위해 `entitySubmit`함수를 사용하고 (예: 새로 생성 된 설문 조사를 나타내는 JSON), 설문 조사 목록을 요청하는 `entityFetch`, 토큰 잔액에 대한 정보를 가져 오기 위해 `accountUpdate`를 사용할 것입니다.

### `frontend/src/view/Index.vue`

기본 양식 구성 요소가 필요하지 않기 때문에 `frontent/src/views/Index.vue` 내부의 `<type-list />`를 새 구성 요소 `<poll-form />`으로 대체하고 새로운 파일 `frontend/src/components/PollForm.vue`을 만들어 작성합니다.

### `frontend/src/components/PollForm.vue`

```javascript
<template>
  <div>
    <app-input placeholder="Title" v-model="title" />
    <div v-for="option in options">
      <app-input placeholder="Option" v-model="option.title" />
    </div>
    <app-button @click.native="add">Add option</app-button>
    <app-button @click.native="submit">Create poll</app-button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      title: "",
      options: []
    };
  },
  methods: {
    add() {
      this.options = [...this.options, { title: "" }];
    },
    async submit() {
      const payload = {
        type: "poll",
        body: {
          title: this.title,
          options: this.options.map(o => o.title)
        }
      };
      await this.$store.dispatch("entitySubmit", payload);
      await this.$store.dispatch("entityFetch", payload);
      await this.$store.dispatch("accountUpdate");
    }
  }
};
</script>
```

`Poll` 폼 컴퍼넌트는 `title`과 `options`의 리스트를 입력 받습니다.

"Add option" 버튼을 클릭하면 빈 입력이 추가되고 "Create poll"을 클릭하면 "create poll" 메시지와 함께 트랜잭션을 전송, 생성 및 브로드 캐스트합니다.

페이지를 새로 고침하고 비밀번호로 로그인 한 다음 새 `poll`을 만듭니다. 트랜잭션을 처리하는 데 몇 초가 걸립니다.

이제 [http://localhost:1317/voter/poll](http://localhost:1317/voter/poll)을 방문하면 투표 목록이 표시됩니다 (이 엔트포인트에 대한 정의는 `x/voter/rest/queryPoll.go`에 있습니다.):
Now, if you visit [http://localhost:1317/voter/poll](http://localhost:1317/voter/poll) you should see a list of polls (this endpoint is defined in `x/voter/rest/queryPoll.go`):

```json
{
  "height": "0",
  "result": [
    {
      "creator": "cosmos19qqa7j73735w4pcx9mkkaxr00af7p432n62tv6",
      "id": "826477ab-0005-4e68-8031-19758d331681",
      "title": "A poll title",
      "options": ["First option", "The second option"]
    }
  ]
}
```

## Adding votes

투표 유형에는 Poll ID와 값 (선택한 옵션의 문자열 표현)이 포함됩니다.

```bash
starport type vote pollID value
```

### `frontend/src/views/Index.vue`

새로운 파일 `frontend/src/components/PollList.vue`에 `<poll-list />` 컴퍼넌트를 생성한 후 `frontend/src/view/Index.vue` 파일의 poll form 컴퍼넌트 뒤에 추가합니다.

### `frontend/src/components/PollList.vue`

```javascript
<template>
  <div>
    <div v-for="poll in polls">
      <app-text type="h2">Poll {{ poll.title }}</app-text>
      <app-radio-item
        @click.native="submit(poll.id, option)"
        v-for="option in poll.options"
        :value="option"
      />
      <app-text type="subtitle">Results: {{ results(poll.id) }}</app-text>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selected: ""
    };
  },
  computed: {
    polls() {
      return this.$store.state.data.poll || [];
    },
    votes() {
      return this.$store.state.data.vote || [];
    }
  },
  methods: {
    results(id) {
      const results = this.votes.filter(v => v.pollID === id);
      return this.$lodash.countBy(results, "value");
    },
    async submit(pollID, value) {
      const type = { type: "vote" };
      const body = { pollID, value };
      await this.$store.dispatch("entitySubmit", { ...type, body });
      await this.$store.dispatch("entityFetch", type);
    }
  }
};
</script>
```

`PollList` 구성 요소는 모든 설문 조사의 모든 옵션을 버튼으로 나열합니다.
The `PollList` component lists for every poll, all the options for that poll, as buttons.

옵션을 선택하면 "create vote" 메시지와 함께 트랜잭션을 브로드 캐스트하고 애플리케이션에서 데이터를 다시 가져 오는 `submit` 메소드가 트리거됩니다.

이제 첫 번째 스크린 샷과 동일한 UI를 볼 수 있습니다.

`poll`을 만들고 투표 해보세요.

하나의 `poll`에 대해 `vote`를 여러번 할 수 있다는 것을 알 수 있습니다.

이것은 우리가 원하는 것이 아니므로이 동작을 수정합시다.

## Casting votes only once

이 문제를 해결하려면 먼저 데이터가 애플리케이션에 저장되는 방식을 이해해야합니다.

데이터를 저장할 때 사전적으로 정렬된 key value store에 저장되었다고 생각할 수 있습니다.

항목을 반복하고, key prefix로 필터링하고, 항목을 추가, 업데이트 및 삭제할 수 있습니다.

저장된 정보를 JSON으로 시각화하는 것이 더 쉽습니다.

```json
{
  "poll-1a266241-c58d-4cbc-bacf-aaf939c95de1": {
    "creator": "cosmos15c6g4v5yvq0hy3kjllyr9ytlx45r936y0m6dm6",
    "id": "1a266241-c58d-4cbc-bacf-aaf939c95de1",
    "title": "Soft drinks",
    "options": ["Coca-Cola", "Pepsi"]
  },
  "vote-cd63b110-2959-45b0-8ce3-afa2fb7a5652": {
    "creator": "cosmos15c6g4v5yvq0hy3kjllyr9ytlx45r936y0m6dm6",
    "id": "cd63b110-2959-45b0-8ce3-afa2fb7a5652",
    "pollID": "1a266241-c58d-4cbc-bacf-aaf939c95de1",
    "value": "Pepsi"
  }
}
```

`poll-`과`vote-`는 모두 prefix 이며, 쉽게 필터링 할 수 있도록 키에 추가됩니다.

prefix는 `x/voter/types/key.go` 에 정의됩니다.

사용자가 투표를 할 때마다 새로운 "create vote" 메시지가 핸들러에 의해 처리되고 keeper에게 전달됩니다.

Keeper는 `vote-` prefix를 사용하고 UUID (모든 메시지에 고유)를 추가하고 이 문자열을 키로 사용합니다.

`x/voter/keeper/vote.go`:

```go
key := []byte(types.VotePrefix + vote.ID)
```

이 문자열은 고유하며 중복 투표를받습니다.

이를 수정하려면 keeper가 올바른 키를 선택하여 모든 투표를 한 번만 기록하도록 해야합니다.

투표 ID와 작성자 주소를 사용하여 한 사용자가 투표 당 한번만 투표 할 수 있도록 할 수 있습니다.

```go
key := []byte(types.VotePrefix + vote.ID + "-" + string(vote.Creator))
```

블록체인을 다시 시작하고 단일 투표에서 여러 번 투표를 시도하면 원하는만큼 여러번 투표 할 수 있지만 가장 최근 투표만 계산됩니다.

## Introducing a fee for creating polls

`poll`을 만드는 데 토큰이 200 개가 들도록 만들어 보겠습니다.

이 기능은 추가하기가 매우 쉽습니다.

먼저 사전에 계정을 등록해야 하며, 각 사용자는 토큰을 보유해야 합니다.

그 다음 `poll`을 만들기 전에 사용자 계정에서 모듈 계정으로 코인을 전송하도록 수정해야 합니다.

## `x/voter/handlerMsgCreatePoll.go`

import 문에 `"[github.com/tendermint/tendermint/crypto](http://github.com/tendermint/tendermint/crypto)"`를 추가하고

`k.CreatePoll(ctx, poll)` 앞에 다음과 같은 코드를 추가합니다.

```go
moduleAcct := sdk.AccAddress(crypto.AddressHash([]byte(types.ModuleName)))
payment, _ := sdk.ParseCoins("200token")
if err := k.CoinKeeper.SendCoins(ctx, poll.Creator, moduleAcct, payment); err != nil {
    return nil, err
}
```

이렇게하면 사용자에게 토큰이 충분하지 않은 경우 블록체인에서 오류가 발생하고 `poll` 생성이 진행되지 않습니다.

이제 앱을 다시 시작하고 여러 설문 조사를 만들어 이것이 토큰 잔액에 어떤 영향을 미치는지 확인할 수 있습니다.
