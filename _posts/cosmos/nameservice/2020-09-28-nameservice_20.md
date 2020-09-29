---
layout: post
title:  "Cosmos-SDK nameservice 20"
author: violetstair
categories: [ CosmosSDK ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: true
---

# Complete App

이제 모듈이 준비 되었으므로 `./app.go`파일에 통합할 차례입니다. nameservice 모듈을 import하여 시작해 보겠습니다.

> _*NOTE*_: 애플리케이션은 방금 작성한 코드를 가져와야 합니다. 여기서 가져오기 경로는 이 저장소(`github.com/cosmos/sdk-tutorials/nameservice/x/nameservice`)로 설정 됩니다. 자신의 저장소에서 작업하는 경우 이를 반영하도록 import 경로를 다음과 같이 변경해야 합니다. (`github.com/{ .Username }/{ .Project.Repo }/x/nameservice`).

```go
package app

import (
    "encoding/json"
    "os"

    abci "github.com/tendermint/tendermint/abci/types"
    "github.com/tendermint/tendermint/libs/log"
    tmos "github.com/tendermint/tendermint/libs/os"
    tmtypes "github.com/tendermint/tendermint/types"
    dbm "github.com/tendermint/tm-db"

    bam "github.com/cosmos/cosmos-sdk/baseapp"
    "github.com/cosmos/cosmos-sdk/codec"
    "github.com/cosmos/cosmos-sdk/simapp"
    sdk "github.com/cosmos/cosmos-sdk/types"
    "github.com/cosmos/cosmos-sdk/types/module"
    "github.com/cosmos/cosmos-sdk/version"
    "github.com/cosmos/cosmos-sdk/x/auth"
    "github.com/cosmos/cosmos-sdk/x/auth/vesting"
    "github.com/cosmos/cosmos-sdk/x/bank"
    distr "github.com/cosmos/cosmos-sdk/x/distribution"
    "github.com/cosmos/cosmos-sdk/x/genutil"
    "github.com/cosmos/cosmos-sdk/x/params"
    "github.com/cosmos/cosmos-sdk/x/slashing"
    "github.com/cosmos/cosmos-sdk/x/staking"
    "github.com/cosmos/cosmos-sdk/x/supply"

    "github.com/cosmos/sdk-tutorials/nameservice/x/nameservice"
)

const appName = "nameservice"

var (
    // DefaultCLIHome : 애플리케이션 CLI의 기본 홈 디렉토리
    DefaultCLIHome = os.ExpandEnv("$HOME/.nscli")

    // DefaultNodeHome : 애플리케이션 데이터 및 설정이 저장되는 디렉토리
    DefaultNodeHome = os.ExpandEnv("$HOME/.nsd")

    // NewBasicManager : 기본 모듈 요소 설정을 담당
    ModuleBasics = module.NewBasicManager(
        genutil.AppModuleBasic{},
        auth.AppModuleBasic{},
        bank.AppModuleBasic{},
        staking.AppModuleBasic{},
        distr.AppModuleBasic{},
        params.AppModuleBasic{},
        slashing.AppModuleBasic{},
        supply.AppModuleBasic{},

        nameservice.AppModule{},
    )
    // 계정 권한
    maccPerms = map[string][]string{
        auth.FeeCollectorName:     nil,
        distr.ModuleName:          nil,
        staking.BondedPoolName:    {supply.Burner, supply.Staking},
        staking.NotBondedPoolName: {supply.Burner, supply.Staking},
    }
)
```

Next you need to add the stores' keys and the `Keepers` into your `nameServiceApp` struct.

```go

type nameServiceApp struct {
    *bam.BaseApp
    cdc *codec.Codec

    // 하위 store에 액세스하기 위한 key
    keys  map[string]*sdk.KVStoreKey
    tkeys map[string]*sdk.TransientStoreKey

    // subspaces
    subspaces map[string]params.Subspace

    // Keepers
    accountKeeper  auth.AccountKeeper
    bankKeeper     bank.Keeper
    stakingKeeper  staking.Keeper
    slashingKeeper slashing.Keeper
    distrKeeper    distr.Keeper
    supplyKeeper   supply.Keeper
    paramsKeeper   params.Keeper
    nsKeeper       nameservice.Keeper

    // Module Manager
    mm *module.Manager

    // simulation manager
    sm *module.SimulationManager
}

// 컴파일 타임에 앱 인터페이스 확인
var _ simapp.App = (*nameServiceApp)(nil)

// NewNameServiceApp : nameServiceApp의 생성자 함수
func NewNameServiceApp(
    logger log.Logger, db dbm.DB, baseAppOptions ...func(*bam.BaseApp),
) *nameServiceApp {

    // 먼저 다른 모듈에서 공유할 최상위 코덱을 정의
    cdc := MakeCodec()

    // BaseApp은 ABCI protocol을 통해 Tendermint와의 상호작용을 처리합니다
    bApp := bam.NewBaseApp(appName, logger, db, auth.DefaultTxDecoder(cdc), baseAppOptions...)

    bApp.SetAppVersion(version.Version)

    keys := sdk.NewKVStoreKeys(bam.MainStoreKey, auth.StoreKey, staking.StoreKey,
        supply.StoreKey, distr.StoreKey, slashing.StoreKey, params.StoreKey, nameservice.StoreKey)

    tkeys := sdk.NewTransientStoreKeys(params.TStoreKey)

    // 여기에서 필요한 store key로 애플리케이션을 초기화 합니다.
    var app = &nameServiceApp{
        BaseApp:   bApp,
        cdc:       cdc,
        keys:      keys,
        tkeys:     tkeys,
        subspaces: make(map[string]params.Subspace),
    }
}
```

이 시점에서 생성자에는 여전히 중요한 로직이 없습니다. 즉 다음과 같은 기능이 필요합니다:

- 원하는 각 모듈에서 필요한 `Keepers`를 인스턴스화 합니다.
- 각 `Keeper`에 필요한 `storeKeys`를 생성합니다.
- 각 모듈에서 `Handler`를 등록합니다. 이를 위해 `baseapp`의 `router`에 있는 `AddRoute()` 메소드가 사용됩니다.
- 각 모듈에서 `Querier`를 등록합니다. 이를 위해 `baseapp`의 `queryRouter`에있는`AddRoute()` 메소드가 사용됩니다.
- `baseApp` 멀티 스토어에서 제공되는 `KVStore`를 마운트 합니다.
- 초기 애플리케이션 상태를 정의하기 위한 `initChainer`를 설정합니다.

완성된 생성자는 다음과 같아야합니다:

```go
// NewNameServiceApp : nameServiceApp의 생성자 함수
func NewNameServiceApp(
    logger log.Logger, db dbm.DB, baseAppOptions ...func(*bam.BaseApp),
) *nameServiceApp {

    // 먼저 다른 모듈에서 공유할 최상위 코덱을 정의합니다
    cdc := MakeCodec()

    // BaseApp은 ABCI protocol을 통해 Tendermint와의 상호작용을 처리합니다
    bApp := bam.NewBaseApp(appName, logger, db, auth.DefaultTxDecoder(cdc), baseAppOptions...)

    bApp.SetAppVersion(version.Version)

    keys := sdk.NewKVStoreKeys(bam.MainStoreKey, auth.StoreKey, staking.StoreKey,
        supply.StoreKey, distr.StoreKey, slashing.StoreKey, params.StoreKey, nameservice.StoreKey)

    tkeys := sdk.NewTransientStoreKeys(params.TStoreKey)

    // 여기에서 필요한 store key로 애플리케이션을 초기화 합니다.
    var app = &nameServiceApp{
        BaseApp:   bApp,
        cdc:       cdc,
        keys:      keys,
        tkeys:     tkeys,
        subspaces: make(map[string]params.Subspace),
    }

    // ParamsKeeper는 애플리케이션에 대한 매개변수 저장을 처리합니다
    app.paramsKeeper = params.NewKeeper(app.cdc, keys[params.StoreKey], tkeys[params.TStoreKey])
    // Set specific supspaces
    app.subspaces[auth.ModuleName] = app.paramsKeeper.Subspace(auth.DefaultParamspace)
    app.subspaces[bank.ModuleName] = app.paramsKeeper.Subspace(bank.DefaultParamspace)
    app.subspaces[staking.ModuleName] = app.paramsKeeper.Subspace(staking.DefaultParamspace)
    app.subspaces[distr.ModuleName] = app.paramsKeeper.Subspace(distr.DefaultParamspace)
    app.subspaces[slashing.ModuleName] = app.paramsKeeper.Subspace(slashing.DefaultParamspace)

    // AccountKeeper handles address -> account 조회
    app.accountKeeper = auth.NewAccountKeeper(
        app.cdc,
        keys[auth.StoreKey],
        app.subspaces[auth.ModuleName],
        auth.ProtoBaseAccount,
    )

    // BankKeeper를 사용하면 sdk.Coins 상호작용을 수행할 수 있습니다
    app.bankKeeper = bank.NewBaseKeeper(
        app.accountKeeper,
        app.subspaces[bank.ModuleName],
        app.ModuleAccountAddrs(),
    )

    // SupplyKeeper는 거래 수수료를 수집하여 수수료 분배 모듈에 랜더링 합니다
    app.supplyKeeper = supply.NewKeeper(
        app.cdc,
        keys[supply.StoreKey],
        app.accountKeeper,
        app.bankKeeper,
        maccPerms,
    )

    // The staking keeper
    stakingKeeper := staking.NewKeeper(
        app.cdc,
        keys[staking.StoreKey],
        app.supplyKeeper,
        app.subspaces[staking.ModuleName],
    )

    app.distrKeeper = distr.NewKeeper(
        app.cdc,
        keys[distr.StoreKey],
        app.subspaces[distr.ModuleName],
        &stakingKeeper,
        app.supplyKeeper,
        auth.FeeCollectorName,
        app.ModuleAccountAddrs(),
    )

    app.slashingKeeper = slashing.NewKeeper(
        app.cdc,
        keys[slashing.StoreKey],
        &stakingKeeper,
        app.subspaces[slashing.ModuleName],
    )

    // staking hooks 등록
    // NOTE: 위의 stakingKeeper는 참조로 전달되므로 이러한 후크를 포함합니다
    app.stakingKeeper = *stakingKeeper.SetHooks(
        staking.NewMultiStakingHooks(
            app.distrKeeper.Hooks(),
            app.slashingKeeper.Hooks()),
    )

    // NameserviceKeeper는 이 튜토리얼 모듈의 Keeper 입니다
    // nameservice의 store와의 상호작용을 처리합니다
    app.nsKeeper = nameservice.NewKeeper(
        app.cdc,
        keys[nameservice.StoreKey],
        app.bankKeeper,
    )

    app.mm = module.NewManager(
        genutil.NewAppModule(app.accountKeeper, app.stakingKeeper, app.BaseApp.DeliverTx),
        auth.NewAppModule(app.accountKeeper),
        bank.NewAppModule(app.bankKeeper, app.accountKeeper),
        nameservice.NewAppModule(app.nsKeeper, app.bankKeeper),
        supply.NewAppModule(app.supplyKeeper, app.accountKeeper),
        distr.NewAppModule(app.distrKeeper, app.accountKeeper, app.supplyKeeper, app.stakingKeeper),
        slashing.NewAppModule(app.slashingKeeper, app.accountKeeper, app.stakingKeeper),
        staking.NewAppModule(app.stakingKeeper, app.accountKeeper, app.supplyKeeper),
    )

    app.mm.SetOrderBeginBlockers(distr.ModuleName, slashing.ModuleName)
    app.mm.SetOrderEndBlockers(staking.ModuleName)

    // Genesis 순서 설정 - 순서가 중요합니다. genutil는 항상 마지막에 옵니다
    // NOTE: genutils 모듈은 staking한 후 발생해야 pool이 genesis 계정의 token으로 제대로 초기화 됩니다
    app.mm.SetOrderInitGenesis(
        distr.ModuleName,
        staking.ModuleName,
        auth.ModuleName,
        bank.ModuleName,
        slashing.ModuleName,
        nameservice.ModuleName,
        supply.ModuleName,
        genutil.ModuleName,
    )

    // 모든 모듈 경로 및 모듈 쿼리 등록
    app.mm.RegisterRoutes(app.Router(), app.QueryRouter())

    // initChainer : genesis.json 파일을 네트워크의 초기 상태로 변환하는 작업을 처리합니다
    app.SetInitChainer(app.InitChainer)
    app.SetBeginBlocker(app.BeginBlocker)
    app.SetEndBlocker(app.EndBlocker)

    // AnteHandler : 서명 확인 및 트랜잭션의 전처리를 담당합니다
    app.SetAnteHandler(
        auth.NewAnteHandler(
            app.accountKeeper,
            app.supplyKeeper,
            auth.DefaultSigVerificationGasConsumer,
        ),
    )

    // initialize stores
    app.MountKVStores(keys)
    app.MountTransientStores(tkeys)

    err := app.LoadLatestVersion(app.keys[bam.MainStoreKey])
    if err != nil {
        tmos.Exit(err.Error())
    }

    return app
}
```

> _*NOTE*_: 위에서 언급한 TransientStore는 지속되지 않는 상태에 대한 KVStore의 in-memory 구현체 입니다

> _*NOTE*_: 모듈이 시작되는 방법에 주의해야 합니다. 순서가 중요합니다. 여기서 시퀀스는 Auth --> Bank --> Feecollection --> Staking --> Distribution --> Slashing으로 이동한 다음 staking 모듈에 대한 후크가 설정되었습니다. 이러한 모듈 중 일부는 사용하기 전에 기존의 다른 모듈에 의존하기 때문입니다.

`initChainer`는 `genesis.json`의 계정이 초기 체인 시작시 애플리케이션 상태에 매핑되는 방식을 정의합니다. `ExportAppStateAndValidators` 함수는 애플리케이션의 초기 상태를 bootstrap하는데 도움이 됩니다. 지금은 이들 중 하나에 대해 너무 많이 걱정할 필요가 없습니다. 또한 우리의 앱에 `BeginBlocker`, `EndBlocker`, `LoadHeight` 메소드를 추가 해야합니다.

생성자는 `initChainer` 함수를 등록하지만 아직 정의되지는 않았습니다. 계속해서 진행해 봅시다

```go
// GenesisState는 체인 시작 부분의 체인 상태를 나타냅니다. 모든 초기상태(account balances)가 여기에 저장됩니다.
type GenesisState map[string]json.RawMessage

func NewDefaultGenesisState() GenesisState {
    return ModuleBasics.DefaultGenesis()
}

func (app *nameServiceApp) InitChainer(ctx sdk.Context, req abci.RequestInitChain) abci.ResponseInitChain {
    var genesisState GenesisState

    err := app.cdc.UnmarshalJSON(req.AppStateBytes, &genesisState)
    if err != nil {
        panic(err)
    }

    return app.mm.InitGenesis(ctx, genesisState)
}

func (app *nameServiceApp) BeginBlocker(ctx sdk.Context, req abci.RequestBeginBlock) abci.ResponseBeginBlock {
    return app.mm.BeginBlock(ctx, req)
}

func (app *nameServiceApp) EndBlocker(ctx sdk.Context, req abci.RequestEndBlock) abci.ResponseEndBlock {
    return app.mm.EndBlock(ctx, req)
}

// GetKey : 제공된 store key에 대한 KVStoreKey 반환
func (app *nameServiceApp) GetKey(storeKey string) *sdk.KVStoreKey {
    return app.keys[storeKey]
}

// GetTKey : 제공된 store key에 대한 TransientStoreKey 반환
func (app *nameServiceApp) GetTKey(storeKey string) *sdk.TransientStoreKey {
    return app.tkeys[storeKey]
}

func (app *nameServiceApp) LoadHeight(height int64) error {
    return app.LoadVersion(height, app.keys[bam.MainStoreKey])
}

// Codec : nameservice의 codec 반환
func (app *nameServiceApp) Codec() *codec.Codec {
    return app.cdc
}

// SimulationManager : SimulationApp 인터페이스 구현
func (app *nameServiceApp) SimulationManager() *module.SimulationManager {
    return app.sm
}

// ModuleAccountAddrs : 모든 앱의 모듈 account address 반환
func (app *nameServiceApp) ModuleAccountAddrs() map[string]bool {
    modAccAddrs := make(map[string]bool)
    for acc := range maccPerms {
        modAccAddrs[supply.NewModuleAddress(acc).String()] = true
    }

    return modAccAddrs
}

//_________________________________________________________

func (app *nameServiceApp) ExportAppStateAndValidators(forZeroHeight bool, jailWhiteList []string,
) (appState json.RawMessage, validators []tmtypes.GenesisValidator, err error) {

    // as if they could withdraw from the start of the next block
    ctx := app.NewContext(true, abci.Header{Height: app.LastBlockHeight()})

    genState := app.mm.ExportGenesis(ctx)
    appState, err = codec.MarshalJSONIndent(app.cdc, genState)
    if err != nil {
        return nil, nil, err
    }

    validators = staking.WriteValidators(ctx, app.stakingKeeper)

    return appState, validators, nil
}
```

마지막으로 애플리케이션에 사용된 모든 모듈을 올바르게 등록하는 [`*codec.Codec`](https://godoc.org/github.com/cosmos/cosmos-sdk/codec#Codec)을 생성하는 helper 함수를 추가합니다

```go
// MakeCodec : Amino에 필요한 코덱을 생성
func MakeCodec() *codec.Codec {
    var cdc = codec.New()

    ModuleBasics.RegisterCodec(cdc)
    vesting.RegisterCodec(cdc)
    sdk.RegisterCodec(cdc)
    codec.RegisterCrypto(cdc)

    return cdc
}
```

## 이제 모듈을 포함하는 애플리케이션을 만들었으므로 entrypoint를 구현할 차례입니다
