# CosmJs

CosmJs 라이브러리를 사용하면 여러분의 체인과 케플러 지갑을 가장 쉽게 연결할 수 있습니다. CosmJs는 타입스크립트/자바스크립트 라이브러리 입니다. 프런트엔드 사용자 인터페이스와 백엔드 서버를 분산 애플리케이션을 구현하는 코스모스 블록체인과 통합할 수 있도록 지원합니다.

![](<../.gitbook/assets/image (1) (1) (1).png>)

CosmJs의 모듈러 구조는 개발자가 필요한 부품만 가져올 수 있도록 해 다운로드 페이로드를 줄이는 데 도움이 됩니다. 라이브러리가 제한이 없기 때문에 Vue, React, Express 와 같은 인기 있는 자바스크립트 프레임워크와 호환됩니다.

CosmJs는 [@cosmjs namespace](https://www.npmjs.com/org/cosmjs) 내에 많은 작은 npm 패키지로 구성된 라이브러리입니다. `stargate` 및 `encoding` 패키지는 Cosmos SDK 체인 버전 0.40 이상과 상호 작용하기 위한 주요 기능을 포함하고 있기 때문에 일반적으로 사람들은 두 개의 패키지만 필요할 것 입니다.

## Get started with CosmJs

### 1. Install modules

아래 모듈을 설치하여 cosmjs를 사용할 수 있습니다.

```javascript
// 대표적인 모듈 예시
npm i @cosmjs/proto-signing
npm i @cosmjs/cosmwasm-stargate
npm i @cosmjs/stargate
npm i @cosmjs/launchpad
```

<대표 간략한 모듈에 대한 설명>

| Module                    | Note                                                             |
| ------------------------- | ---------------------------------------------------------------- |
| @cosmjs/proto-signing     | dApp 프론트엔드를 사용자 지갑에 연결하기 위해 protobuf 기반 메시지 서명을 위한 유틸리티가 포함된 패키지 |
| @cosmjs/stargate          | Cosmos SDK 용 클라이언트 라이브러리                                         |
| @cosmjs/cosmwasm-stargate | CosmWasm 스마트 계약 을 위한 기능으로 확장된 라이브러리                              |

이 외 CosmJs 모듈과 자세한 설명은 [해당 링크](https://www.npmjs.com/org/cosmjs)에서 확인할 수 있습니다.

### 2. How to detect Keplr

`window.keplr` 를 확인하여 사용자 디바이스에 Keplr가 설치되어 있는지 여부를 확인할 수 있습니다. `window.keplr`가 document.load 이후에 `undefined` 를 반환하는 경우, Keplr 설치되지 않습니다. 로드 이벤트가 상태를 확인할 때까지 기다리는 방법은 여러 가지가 있습니다. 아래 예를 참조하십시오.

`window.onload`에 함수를 등록할 수 있습니다.



`OfflineSigner` 를 사용해서 케플러를 CosmJs에 연결할 수 있습니다.

```javascript
import { SigningCosmosClient } from '@cosmjs/launchpad'

window.onload = async () => {
    if (!window.keplr) {
        alert("Please install keplr extension");
    } else {
        const chainId = "cosmoshub-4"; //Cosmos Hub chainId
        
        // Enabling before using the Keplr is recommended.
        // This method will ask the user whether to allow access if they haven't visited this website.
        // Also, it will request that the user unlock the wallet if the wallet is locked.
        await window.keplr.enable(chainId);

        const offlineSigner = window.keplr.getOfflineSigner(chainId);

        // You can get the address/public keys by `getAccounts` method.
        // It can return the array of address/public key.
        // But, currently, Keplr extension manages only one address/public key pair.
        // XXX: This line is needed to set the sender address for SigningCosmosClient.
        const accounts = await offlineSigner.getAccounts();

        // Initialize the gaia api with the offline signer that is injected by Keplr extension.
        const cosmJS = new SigningCosmosClient(
            "https://lcd-cosmoshub.keplr.app", //Cosmos Hub RPC
            accounts[0].address,
            offlineSigner,
        );
    }
}
```

`OfflineSigner` 를 가져오려면, `keplr.getOfflineSigner(chainId)` 또는 `window.getOfflineSigner(chainId)`를 사용합니다. ( `window.getOfflineSigner` `keplr.getOfflineSigner`를 실행하고 값을 반환하는 별칭입니다.)

`window.keplr.enable(chainId)` 메서드는 Keplr 확장자가 잠겨 있으면 사용자에게 그들의 케플러 익스텐션 잠금을 해제하도록 요청합니다. 사용자가 익스텐션을 웹 사이트에 연결할 수 있는 권한을 부여하지 않은 경우 먼저 웹 사이트 연결을 요청하게 됩니다.

사용자가 잠금 해제를 취소하거나 연결 권한을 거부하면 오류가 발생합니다.

만약 익스텐션의 잠금이 이미 해제되어 있고 웹 사이트에 연결할 수 있는 권한이 있는 경우 아무 문제없이 해결됩니다.

`window.keplr.enable(chainId)`는 필수 항목이 아닙니다. 메서드가 호출되지 않았더라도 Keplr에 대한 액세스를 요청하는 API가 호출되면 위의 흐름이 자동으로 실행됩니다. 그러나 `window.keplr.enable(chainId)`을 처음 실행하는 것이 좋습니다.

<details>

<summary>Types of Offline Signers</summary>

In CosmJS, there are two types of Signers: OfflineSigner and OfflineDirectSigner. OfflineSigner is used to sign SignDoc serialized with Amino in Cosmos SDK Launchpad (Cosmos SDK v0.39.x or below). OfflineDirectSigner is used to sign Protobuf encoded SignDoc.

Keplr supports both types of Signers. Keplr’s `keplr.getOfflineSigner(chainId)` or `window.getOfflineSigner(chainId)` returns a Signer that satisfies both the OfflineSigner and OfflineDirectSigner. Therefore, when using CosmJS with this Signer, Amino is used for Launchpad chains and Protobuf is used for Stargate chains.

However, if the msg to be sent is able to be serialized/deserialized using Amino codec you can use a signer for Amino. Also, as there are some limitations to protobuf type sign doc, there may be cases when Amino is necessary. For example, Protobuf formatted sign doc is currently not supported by Ledger Nano’s Cosmos app. Also, because protobuf sign doc is binary formatted, msgs not natively supported by Keplr may not be human-readable.

If you’d like to enforce the use of Amino, you can use the following APIs: `keplr.getOfflineSignerOnlyAmino(chainId)` or `window.getOfflineSignerOnlyAmino(chainId: string)`. Because this will always return an Amino compatible signer, any CosmJS requested msg that is Amino compatible will request an Amino SignDoc to Keplr.

Also, `window.getOfflineSignerAuto(chainId: string): Promise<OfflineSigner | OfflineDirectSigner>` or `window.getOfflineSignerAuto(chainId: string): Promise<OfflineSigner | OfflineDirectSigner>` API is supported. Please note that the value returned is async. This API automatically returns a signer that only supports Amino if the account is a Ledger-based account, and returns a signer that is compatible for both Amino and Protobuf if the account is a mnemonic/private key-based account. Because this API is affected by the type of the connected Keplr account, if [keplr\_keystorechange](https://docs.keplr.app/api/#change-key-store-event) event is used to detect account changes the signer must be changed using the API when this event has been triggered.

</details>

### 3. Connect to network

먼저 아래와 같은 명령어를 실행하여 모듈을 다운 받아주세요.

```javascript
npm i @keplr-wallet/cosmos
```

이용하기에 앞서서 연결하고자 하는 네트워크, 체인에 대한 정보를 기입해야합니다. 각각의 체인에 대한 정보 양식은 아래를 참조하여 사용할 수 있습니다.

<체인이름 별 정보 양식>

<details>

<summary>Osmosis</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos =
	{
	rpc: 'https://rpc-osmosis.blockapsis.com',
  	rest: 'https://lcd-osmosis.blockapsis.com',
	chainId: 'osmosis-1',
	chainName: 'Osmosis',
	stakeCurrency: {
		coinDenom: 'OSMO',
		coinMinimalDenom: 'uosmo',
		coinDecimals: 6,
		coinGeckoId: 'osmosis',
		coinImageUrl: window.location.origin + '/public/assets/tokens/osmosis.svg',
	},
	bip44: {
		coinType: 118,
	},
		bech32Config: Bech32Address.defaultBech32Config('osmo'),
		currencies: [
			{
				coinDenom: 'OSMO',
				coinMinimalDenom: 'uosmo',
				coinDecimals: 6,
				coinGeckoId: 'osmosis',
				coinImageUrl: window.location.origin + '/public/assets/tokens/osmosis.svg',
			},
			{
				coinDenom: 'ION',
				coinMinimalDenom: 'uion',
				coinDecimals: 6,
				coinGeckoId: 'ion',
				coinImageUrl: window.location.origin + '/public/assets/tokens/ion.png',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'OSMO',
				coinMinimalDenom: 'uosmo',
				coinDecimals: 6,
				coinGeckoId: 'osmosis',
				coinImageUrl: window.location.origin + '/public/assets/tokens/osmosis.svg',
			},
		],
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx', 'ibc-go'],
		explorerUrlToTx: 'https://www.mintscan.io/osmosis/txs/{txHash}',
	}
```

</details>

<details>

<summary>Cosmos Hub</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-cosmoshub.keplr.app',
		rest: 'https://lcd-cosmoshub.keplr.app',
		chainId: 'cosmoshub-4',
		chainName: 'Cosmos Hub',
		stakeCurrency: {
			coinDenom: 'ATOM',
			coinMinimalDenom: 'uatom',
			coinDecimals: 6,
			coinGeckoId: 'cosmos',
			coinImageUrl: window.location.origin + '/public/assets/tokens/cosmos.svg',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('cosmos'),
		currencies: [
			{
				coinDenom: 'ATOM',
				coinMinimalDenom: 'uatom',
				coinDecimals: 6,
				coinGeckoId: 'cosmos',
				coinImageUrl: window.location.origin + '/public/assets/tokens/cosmos.svg',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'ATOM',
				coinMinimalDenom: 'uatom',
				coinDecimals: 6,
				coinGeckoId: 'cosmos',
				coinImageUrl: window.location.origin + '/public/assets/tokens/cosmos.svg',
			},
		],
		coinType: 118,
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx', 'ibc-go'],
		explorerUrlToTx: 'https://www.mintscan.io/cosmos/txs/{txHash}',
	},
```

</details>

<details>

<summary>Terra</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
		{
		rpc: 'https://rpc-columbus.keplr.app',
		rest: 'https://lcd-columbus.keplr.app',
		chainId: 'columbus-5',
		chainName: 'Terra',
		stakeCurrency: {
			coinDenom: 'LUNA',
			coinMinimalDenom: 'uluna',
			coinDecimals: 6,
			coinGeckoId: 'terra-luna',
			coinImageUrl: window.location.origin + '/public/assets/tokens/luna.png',
		},
		bip44: {
			coinType: 330,
		},
		bech32Config: Bech32Address.defaultBech32Config('terra'),
		currencies: [
			{
				coinDenom: 'LUNA',
				coinMinimalDenom: 'uluna',
				coinDecimals: 6,
				coinGeckoId: 'terra-luna',
				coinImageUrl: window.location.origin + '/public/assets/tokens/luna.png',
			},
			{
				coinDenom: 'UST',
				coinMinimalDenom: 'uusd',
				coinDecimals: 6,
				coinGeckoId: 'terrausd',
				coinImageUrl: window.location.origin + '/public/assets/tokens/ust.png',
			},
			{
				coinDenom: 'KRT',
				coinMinimalDenom: 'ukrw',
				coinDecimals: 6,
				coinGeckoId: 'terra-krw',
				coinImageUrl: window.location.origin + '/public/assets/tokens/krt.png',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'LUNA',
				coinMinimalDenom: 'uluna',
				coinDecimals: 6,
				coinGeckoId: 'terra-luna',
				coinImageUrl: window.location.origin + '/public/assets/tokens/luna.png',
			},
			{
				coinDenom: 'UST',
				coinMinimalDenom: 'uusd',
				coinDecimals: 6,
				coinGeckoId: 'terrausd',
				coinImageUrl: window.location.origin + '/public/assets/tokens/ust.png',
			},
		],
		gasPriceStep: {
			low: 0.015,
			average: 0.015,
			high: 0.015,
		},
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx'],
		explorerUrlToTx: 'https://finder.terra.money/columbus-5/tx/{txHash}',
	}
```

</details>

<details>

<summary>Secret Network</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-secret.keplr.app',
		rest: 'https://lcd-secret.keplr.app',
		chainId: 'secret-4',
		chainName: 'Secret Network',
		stakeCurrency: {
			coinDenom: 'SCRT',
			coinMinimalDenom: 'uscrt',
			coinDecimals: 6,
			coinGeckoId: 'secret',
			coinImageUrl: window.location.origin + '/public/assets/tokens/scrt.svg',
		},
		bip44: {
			coinType: 529,
		},
		bech32Config: Bech32Address.defaultBech32Config('secret'),
		currencies: [
			{
				coinDenom: 'SCRT',
				coinMinimalDenom: 'uscrt',
				coinDecimals: 6,
				coinGeckoId: 'secret',
				coinImageUrl: window.location.origin + '/public/assets/tokens/scrt.svg',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'SCRT',
				coinMinimalDenom: 'uscrt',
				coinDecimals: 6,
				coinGeckoId: 'secret',
				coinImageUrl: window.location.origin + '/public/assets/tokens/scrt.svg',
			},
		],
		coinType: 118,
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx'],
		explorerUrlToTx: 'https://secretnodes.com/secret/chains/secret-4/transactions/{txHash}',
	}
```

</details>

<details>

<summary>Akash</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-akash.keplr.app',
		rest: 'https://lcd-akash.keplr.app',
		chainId: 'akashnet-2',
		chainName: 'Akash',
		stakeCurrency: {
			coinDenom: 'AKT',
			coinMinimalDenom: 'uakt',
			coinDecimals: 6,
			coinGeckoId: 'akash-network',
			coinImageUrl: window.location.origin + '/public/assets/tokens/akt.svg',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('akash'),
		currencies: [
			{
				coinDenom: 'AKT',
				coinMinimalDenom: 'uakt',
				coinDecimals: 6,
				coinGeckoId: 'akash-network',
				coinImageUrl: window.location.origin + '/public/assets/tokens/akt.svg',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'AKT',
				coinMinimalDenom: 'uakt',
				coinDecimals: 6,
				coinGeckoId: 'akash-network',
				coinImageUrl: window.location.origin + '/public/assets/tokens/akt.svg',
			},
		],
		coinType: 118,
		features: ['stargate', 'ibc-transfer'],
		explorerUrlToTx: 'https://www.mintscan.io/akash/txs/{txHash}',
	}
```

</details>

<details>

<summary>Regen Network</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-regen.keplr.app',
		rest: 'https://lcd-regen.keplr.app',
		chainId: 'regen-1',
		chainName: 'Regen Network',
		stakeCurrency: {
			coinDenom: 'REGEN',
			coinMinimalDenom: 'uregen',
			coinDecimals: 6,
			coinImageUrl: window.location.origin + '/public/assets/tokens/regen.png',
			coinGeckoId: 'regen',
		},
		bip44: { coinType: 118 },
		bech32Config: Bech32Address.defaultBech32Config('regen'),
		currencies: [
			{
				coinDenom: 'REGEN',
				coinMinimalDenom: 'uregen',
				coinDecimals: 6,
				coinImageUrl: window.location.origin + '/public/assets/tokens/regen.png',
				coinGeckoId: 'regen',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'REGEN',
				coinMinimalDenom: 'uregen',
				coinDecimals: 6,
				coinImageUrl: window.location.origin + '/public/assets/tokens/regen.png',
				coinGeckoId: 'regen',
			},
		],
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx'],
		explorerUrlToTx: 'https://regen.aneka.io/txs/{txHash}',
	}
```

</details>

<details>

<summary>Sentinel</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-sentinel.keplr.app',
		rest: 'https://lcd-sentinel.keplr.app',
		chainId: 'sentinelhub-2',
		chainName: 'Sentinel',
		stakeCurrency: {
			coinDenom: 'DVPN',
			coinMinimalDenom: 'udvpn',
			coinDecimals: 6,
			coinGeckoId: 'sentinel',
			coinImageUrl: window.location.origin + '/public/assets/tokens/dvpn.png',
		},
		bip44: { coinType: 118 },
		bech32Config: Bech32Address.defaultBech32Config('sent'),
		currencies: [
			{
				coinDenom: 'DVPN',
				coinMinimalDenom: 'udvpn',
				coinDecimals: 6,
				coinGeckoId: 'sentinel',
				coinImageUrl: window.location.origin + '/public/assets/tokens/dvpn.png',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'DVPN',
				coinMinimalDenom: 'udvpn',
				coinDecimals: 6,
				coinGeckoId: 'sentinel',
				coinImageUrl: window.location.origin + '/public/assets/tokens/dvpn.png',
			},
		],
		explorerUrlToTx: 'https://www.mintscan.io/sentinel/txs/{txHash}',
		features: ['stargate', 'ibc-transfer'],
	}
```

</details>

<details>

<summary>Persistence</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-persistence.keplr.app',
		rest: 'https://lcd-persistence.keplr.app',
		chainId: 'core-1',
		chainName: 'Persistence',
		stakeCurrency: {
			coinDenom: 'XPRT',
			coinMinimalDenom: 'uxprt',
			coinDecimals: 6,
			coinGeckoId: 'persistence',
			coinImageUrl: window.location.origin + '/public/assets/tokens/xprt.png',
		},
		bip44: {
			coinType: 750,
		},
		bech32Config: Bech32Address.defaultBech32Config('persistence'),
		currencies: [
			{
				coinDenom: 'XPRT',
				coinMinimalDenom: 'uxprt',
				coinDecimals: 6,
				coinGeckoId: 'persistence',
				coinImageUrl: window.location.origin + '/public/assets/tokens/xprt.png',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'XPRT',
				coinMinimalDenom: 'uxprt',
				coinDecimals: 6,
				coinGeckoId: 'persistence',
				coinImageUrl: window.location.origin + '/public/assets/tokens/xprt.png',
			},
		],
		features: ['stargate', 'ibc-transfer'],
		explorerUrlToTx: 'https://www.mintscan.io/persistence/txs/{txHash}',
	}
```

</details>

<details>

<summary>IRISnet</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-iris.keplr.app',
		rest: 'https://lcd-iris.keplr.app',
		chainId: 'irishub-1',
		chainName: 'IRISnet',
		stakeCurrency: {
			coinDenom: 'IRIS',
			coinMinimalDenom: 'uiris',
			coinDecimals: 6,
			coinGeckoId: 'iris-network',
			coinImageUrl: window.location.origin + '/public/assets/tokens/iris.svg',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('iaa'),
		currencies: [
			{
				coinDenom: 'IRIS',
				coinMinimalDenom: 'uiris',
				coinDecimals: 6,
				coinGeckoId: 'iris-network',
				coinImageUrl: window.location.origin + '/public/assets/tokens/iris.svg',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'IRIS',
				coinMinimalDenom: 'uiris',
				coinDecimals: 6,
				coinGeckoId: 'iris-network',
				coinImageUrl: window.location.origin + '/public/assets/tokens/iris.svg',
			},
		],
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx'],
		explorerUrlToTx: 'https://www.mintscan.io/iris/txs/{txHash}'
	}
```

</details>

<details>

<summary>Crypto.org</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-crypto-org.keplr.app/',
		rest: 'https://lcd-crypto-org.keplr.app/',
		chainId: 'crypto-org-chain-mainnet-1',
		chainName: 'Crypto.org',
		stakeCurrency: {
			coinDenom: 'CRO',
			coinMinimalDenom: 'basecro',
			coinDecimals: 8,
			coinGeckoId: 'crypto-com-chain',
			coinImageUrl: window.location.origin + '/public/assets/tokens/cro.png',
		},
		bip44: {
			coinType: 394,
		},
		bech32Config: Bech32Address.defaultBech32Config('cro'),
		currencies: [
			{
				coinDenom: 'CRO',
				coinMinimalDenom: 'basecro',
				coinDecimals: 8,
				coinGeckoId: 'crypto-com-chain',
				coinImageUrl: window.location.origin + '/public/assets/tokens/cro.png',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'CRO',
				coinMinimalDenom: 'basecro',
				coinDecimals: 8,
				coinGeckoId: 'crypto-com-chain',
				coinImageUrl: window.location.origin + '/public/assets/tokens/cro.png',
			},
		],
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx'],
		explorerUrlToTx: 'https://www.mintscan.io/crypto-org/txs/{txHash}',
	}
```

</details>

<details>

<summary>Starname</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-iov.keplr.app',
		rest: 'https://lcd-iov.keplr.app',
		chainId: 'iov-mainnet-ibc',
		chainName: 'Starname',
		stakeCurrency: {
			coinDenom: 'IOV',
			coinMinimalDenom: 'uiov',
			coinDecimals: 6,
			coinGeckoId: 'starname',
			coinImageUrl: window.location.origin + '/public/assets/tokens/iov.png',
		},
		bip44: {
			coinType: 234,
		},
		bech32Config: Bech32Address.defaultBech32Config('star'),
		currencies: [
			{
				coinDenom: 'IOV',
				coinMinimalDenom: 'uiov',
				coinDecimals: 6,
				coinGeckoId: 'starname',
				coinImageUrl: window.location.origin + '/public/assets/tokens/iov.png',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'IOV',
				coinMinimalDenom: 'uiov',
				coinDecimals: 6,
				coinGeckoId: 'starname',
				coinImageUrl: window.location.origin + '/public/assets/tokens/iov.png',
			},
		],
		features: ['stargate', 'ibc-transfer'],
		explorerUrlToTx: 'https://www.mintscan.io/starname/txs/{txHash}',
	}
```

</details>

<details>

<summary>e-Money</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-emoney.keplr.app',
		rest: 'https://lcd-emoney.keplr.app',
		chainId: 'emoney-3',
		chainName: 'e-Money',
		stakeCurrency: {
			coinDenom: 'NGM',
			coinMinimalDenom: 'ungm',
			coinDecimals: 6,
			coinGeckoId: 'e-money',
			coinImageUrl: window.location.origin + '/public/assets/tokens/ngm.png',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('emoney'),
		currencies: [
			{
				coinDenom: 'NGM',
				coinMinimalDenom: 'ungm',
				coinDecimals: 6,
				coinGeckoId: 'e-money',
				coinImageUrl: window.location.origin + '/public/assets/tokens/ngm.png',
			},
			{
				coinDenom: 'EEUR',
				coinMinimalDenom: 'eeur',
				coinDecimals: 6,
				coinGeckoId: 'e-money-eur',
				coinImageUrl: window.location.origin + '/public/assets/tokens/eeur.png',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'NGM',
				coinMinimalDenom: 'ungm',
				coinDecimals: 6,
				coinGeckoId: 'e-money',
				coinImageUrl: window.location.origin + '/public/assets/tokens/ngm.png',
			},
		],
		gasPriceStep: {
			low: 1,
			average: 1,
			high: 1,
		},
		features: ['stargate', 'ibc-transfer'],
		explorerUrlToTx: 'https://emoney.bigdipper.live/transactions/{txHash}',
	}
```

</details>

<details>

<summary>Juno</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-juno.keplr.app',
		rest: 'https://lcd-juno.keplr.app',
		chainId: 'juno-1',
		chainName: 'Juno',
		stakeCurrency: {
			coinDenom: 'JUNO',
			coinMinimalDenom: 'ujuno',
			coinDecimals: 6,
			coinGeckoId: 'juno-network',
			coinImageUrl: window.location.origin + '/public/assets/tokens/juno.svg',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('juno'),
		currencies: [
			{
				coinDenom: 'JUNO',
				coinMinimalDenom: 'ujuno',
				coinDecimals: 6,
				coinGeckoId: 'juno-network',
				coinImageUrl: window.location.origin + '/public/assets/tokens/juno.svg',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'JUNO',
				coinMinimalDenom: 'ujuno',
				coinDecimals: 6,
				coinGeckoId: 'juno-network',
				coinImageUrl: window.location.origin + '/public/assets/tokens/juno.svg',
			},
		],
		features: ['stargate', 'ibc-transfer'],
		explorerUrlToTx: 'https://www.mintscan.io/juno/txs/{txHash}',
	}
```

</details>

<details>

<summary>Juno Uni Testnet</summary>

```javascript
import { Bech32Address } from "@keplr-wallet/cosmos";

const ChainInfo = {
  rpc: "https://rpc.uni.junomint.com",
  rest: "https://lcd-juno.keplr.app",
  chainId: "uni-3",
  chainName: "Juno Uni Testnet",
  stakeCurrency: {
    coinDenom: "JUNOX",
    coinMinimalDenom: "ujunox",
    coinDecimals: 6,
    coinGeckoId: "juno-network",
    coinImageUrl: window.location.origin + "/public/assets/tokens/juno.svg",
  },
  bip44: {
    coinType: 118,
  },
  bech32Config: Bech32Address.defaultBech32Config("juno"),
  currencies: [
    {
      coinDenom: "JUNOX",
      coinMinimalDenom: "ujunox",
      coinDecimals: 6,
      coinGeckoId: "juno-network",
      coinImageUrl: window.location.origin + "/public/assets/tokens/juno.svg",
    },
  ],
  feeCurrencies: [
    {
      coinDenom: "JUNOX",
      coinMinimalDenom: "ujunox",
      coinDecimals: 6,
      coinGeckoId: "juno-network",
      coinImageUrl: window.location.origin + "/public/assets/tokens/juno.svg",
    },
  ],
  features: ["stargate", "ibc-transfer"],
  explorerUrlToTx: "https://www.mintscan.io/juno/txs/{txHash}",
};

export default ChainInfo;
```

</details>

<details>

<summary>Microtick</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-microtick.keplr.app',
		rest: 'https://lcd-microtick.keplr.app',
		chainId: 'microtick-1',
		chainName: 'Microtick',
		stakeCurrency: {
			coinDenom: 'TICK',
			coinMinimalDenom: 'utick',
			coinDecimals: 6,
			coinGeckoId: 'pool:utick',
			coinImageUrl: window.location.origin + '/public/assets/tokens/tick.svg',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('micro'),
		currencies: [
			{
				coinDenom: 'TICK',
				coinMinimalDenom: 'utick',
				coinDecimals: 6,
				coinGeckoId: 'pool:utick',
				coinImageUrl: window.location.origin + '/public/assets/tokens/tick.svg',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'TICK',
				coinMinimalDenom: 'utick',
				coinDecimals: 6,
				coinGeckoId: 'pool:utick',
				coinImageUrl: window.location.origin + '/public/assets/tokens/tick.svg',
			},
		],
		features: ['stargate', 'ibc-transfer'],
		explorerUrlToTx: 'https://explorer.microtick.zone/transactions/{txHash}',
	}
```

</details>

<details>

<summary>LikeCoin</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://mainnet-node.like.co/rpc',
		rest: 'https://mainnet-node.like.co',
		chainId: 'likecoin-mainnet-2',
		chainName: 'LikeCoin',
		stakeCurrency: {
			coinDenom: 'LIKE',
			coinMinimalDenom: 'nanolike',
			coinDecimals: 9,
			coinGeckoId: 'likecoin',
			coinImageUrl: window.location.origin + '/public/assets/tokens/likecoin.svg',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('cosmos'),
		currencies: [
			{
				coinDenom: 'LIKE',
				coinMinimalDenom: 'nanolike',
				coinDecimals: 9,
				coinGeckoId: 'likecoin',
				coinImageUrl: window.location.origin + '/public/assets/tokens/likecoin.svg',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'LIKE',
				coinMinimalDenom: 'nanolike',
				coinDecimals: 9,
				coinGeckoId: 'likecoin',
				coinImageUrl: window.location.origin + '/public/assets/tokens/likecoin.svg',
			},
		],
		features: ['stargate', 'ibc-transfer'],
		explorerUrlToTx: 'https://likecoin.bigdipper.live/transactions/{txHash}',
	}
```

</details>

<details>

<summary>IXO</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-impacthub.keplr.app',
		rest: 'https://lcd-impacthub.keplr.app',
		chainId: 'impacthub-3',
		chainName: 'IXO',
		stakeCurrency: {
			coinDenom: 'IXO',
			coinMinimalDenom: 'uixo',
			coinDecimals: 6,
			coinGeckoId: 'pool:uixo',
			coinImageUrl: window.location.origin + '/public/assets/tokens/ixo.svg',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('ixo'),
		currencies: [
			{
				coinDenom: 'IXO',
				coinMinimalDenom: 'uixo',
				coinDecimals: 6,
				coinGeckoId: 'pool:uixo',
				coinImageUrl: window.location.origin + '/public/assets/tokens/ixo.png',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'IXO',
				coinMinimalDenom: 'uixo',
				coinDecimals: 6,
				coinGeckoId: 'pool:uixo',
				coinImageUrl: window.location.origin + '/public/assets/tokens/ixo.png',
			},
		],
		features: ['stargate', 'ibc-transfer'],
		explorerUrlToTx: 'https://blockscan.ixo.world/transactions/{txHash}',
	}
```

</details>

<details>

<summary>BitCanna</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc.bitcanna.io',
		rest: 'https://lcd.bitcanna.io',
		chainId: 'bitcanna-1',
		chainName: 'BitCanna',
		stakeCurrency: {
			coinDenom: 'BCNA',
			coinMinimalDenom: 'ubcna',
			coinDecimals: 6,
			coinGeckoId: 'bitcanna',
			coinImageUrl: window.location.origin + '/public/assets/tokens/bcna.svg',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('bcna'),
		currencies: [
			{
				coinDenom: 'BCNA',
				coinMinimalDenom: 'ubcna',
				coinDecimals: 6,
				coinGeckoId: 'bitcanna',
				coinImageUrl: window.location.origin + '/public/assets/tokens/bcna.svg',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'BCNA',
				coinMinimalDenom: 'ubcna',
				coinDecimals: 6,
				coinGeckoId: 'bitcanna',
				coinImageUrl: window.location.origin + '/public/assets/tokens/bcna.svg',
			},
		],
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx'],
		explorerUrlToTx: 'https://www.mintscan.io/bitcanna/txs/{txHash}',
	}
```

</details>

<details>

<summary>BitSong</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc.explorebitsong.com',
		rest: 'https://lcd.explorebitsong.com',
		chainId: 'bitsong-2b',
		chainName: 'BitSong',
		stakeCurrency: {
			coinDenom: 'BTSG',
			coinMinimalDenom: 'ubtsg',
			coinDecimals: 6,
			coinGeckoId: 'pool:ubtsg',
			coinImageUrl: window.location.origin + '/public/assets/tokens/btsg.svg',
		},
		bip44: {
			coinType: 639,
		},
		bech32Config: Bech32Address.defaultBech32Config('bitsong'),
		currencies: [
			{
				coinDenom: 'BTSG',
				coinMinimalDenom: 'ubtsg',
				coinDecimals: 6,
				coinGeckoId: 'pool:ubtsg',
				coinImageUrl: window.location.origin + '/public/assets/tokens/btsg.svg',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'BTSG',
				coinMinimalDenom: 'ubtsg',
				coinDecimals: 6,
				coinGeckoId: 'pool:ubtsg',
				coinImageUrl: window.location.origin + '/public/assets/tokens/btsg.svg',
			},
		],
		features: ['stargate', 'ibc-transfer'],
		explorerUrlToTx: 'https://explorebitsong.com/transactions/{txHash}',
	}
```

</details>

<details>

<summary>kichain-2</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-mainnet.blockchain.ki',
		rest: 'https://api-mainnet.blockchain.ki',
		chainId: 'kichain-2',
		chainName: 'Ki',
		stakeCurrency: {
			coinDenom: 'XKI',
			coinMinimalDenom: 'uxki',
			coinDecimals: 6,
			coinGeckoId: 'pool:uxki',
			coinImageUrl: window.location.origin + '/public/assets/tokens/ki.svg',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('ki'),
		currencies: [
			{
				coinDenom: 'XKI',
				coinMinimalDenom: 'uxki',
				coinDecimals: 6,
				coinGeckoId: 'pool:uxki',
				coinImageUrl: window.location.origin + '/public/assets/tokens/ki.svg',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'XKI',
				coinMinimalDenom: 'uxki',
				coinDecimals: 6,
				coinGeckoId: 'pool:uxki',
				coinImageUrl: window.location.origin + '/public/assets/tokens/ki.svg',
			},
		],
		features: ['stargate', 'ibc-transfer'],
		explorerUrlToTx: 'https://www.mintscan.io/ki-chain/txs/{txHash}',
	}
```

</details>

<details>

<summary>MediBloc</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc.gopanacea.org',
		rest: 'https://api.gopanacea.org',
		chainId: 'panacea-3',
		chainName: 'MediBloc',
		stakeCurrency: {
			coinDenom: 'MED',
			coinMinimalDenom: 'umed',
			coinDecimals: 6,
			coinGeckoId: 'medibloc',
			coinImageUrl: window.location.origin + '/public/assets/tokens/med.png',
		},
		bip44: {
			coinType: 371,
		},
		bech32Config: Bech32Address.defaultBech32Config('panacea'),
		currencies: [
			{
				coinDenom: 'MED',
				coinMinimalDenom: 'umed',
				coinDecimals: 6,
				coinGeckoId: 'medibloc',
				coinImageUrl: window.location.origin + '/public/assets/tokens/med.png',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'MED',
				coinMinimalDenom: 'umed',
				coinDecimals: 6,
				coinGeckoId: 'medibloc',
				coinImageUrl: window.location.origin + '/public/assets/tokens/med.png',
			},
		],
		gasPriceStep: {
			low: 5,
			average: 7,
			high: 9,
		},
		features: ['stargate', 'ibc-transfer'],
		explorerUrlToTx: 'https://www.mintscan.io/medibloc/txs/{txHash}',
	}
```

</details>

<details>

<summary>Bostrom</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc.bostrom.cybernode.ai',
		rest: 'https://lcd.bostrom.cybernode.ai',
		chainId: 'bostrom',
		chainName: 'Bostrom',
		stakeCurrency: {
			coinDenom: 'BOOT',
			coinMinimalDenom: 'boot',
			coinDecimals: 0,
			// coinGeckoId: 'pool:boot',
			coinImageUrl: window.location.origin + '/public/assets/tokens/boot.png',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('bostrom'),
		currencies: [
			{
				coinDenom: 'BOOT',
				coinMinimalDenom: 'boot',
				coinDecimals: 0,
				// coinGeckoId: 'pool:boot',
				coinImageUrl: window.location.origin + '/public/assets/tokens/boot.png',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'BOOT',
				coinMinimalDenom: 'boot',
				coinDecimals: 0,
				// coinGeckoId: 'pool:boot',
				coinImageUrl: window.location.origin + '/public/assets/tokens/boot.png',
			},
		],
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx'],
		explorerUrlToTx: 'https://cyb.ai/network/bostrom/tx/{txHash}',
	}
```

</details>

<details>

<summary>Comdex</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc.comdex.one',
		rest: 'https://rest.comdex.one',
		chainId: 'comdex-1',
		chainName: 'Comdex',
		stakeCurrency: {
			coinDenom: 'CMDX',
			coinMinimalDenom: 'ucmdx',
			coinDecimals: 6,
			coinGeckoId: 'comdex',
			coinImageUrl: window.location.origin + '/public/assets/tokens/cmdx.png',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('comdex'),
		currencies: [
			{
				coinDenom: 'CMDX',
				coinMinimalDenom: 'ucmdx',
				coinDecimals: 6,
				coinGeckoId: 'comdex',
				coinImageUrl: window.location.origin + '/public/assets/tokens/cmdx.png',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'CMDX',
				coinMinimalDenom: 'ucmdx',
				coinDecimals: 6,
				coinGeckoId: 'comdex',
				coinImageUrl: window.location.origin + '/public/assets/tokens/cmdx.png',
			},
		],
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx'],
		explorerUrlToTx: 'https://www.mintscan.io/comdex/txs/{txHash}',
	}
```

</details>

<details>

<summary>cheqd-mainnet-1</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc.cheqd.net',
		rest: 'https://api.cheqd.net',
		chainId: 'cheqd-mainnet-1',
		chainName: 'cheqd',
		stakeCurrency: {
			coinDenom: 'CHEQ',
			coinMinimalDenom: 'ncheq',
			coinDecimals: 9,
			coinGeckoId: 'cheqd-network',
			coinImageUrl: window.location.origin + '/public/assets/tokens/cheq.svg',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('cheqd'),
		currencies: [
			{
				coinDenom: 'CHEQ',
				coinMinimalDenom: 'ncheq',
				coinDecimals: 9,
				coinGeckoId: 'cheqd-network',
				coinImageUrl: window.location.origin + '/public/assets/tokens/cheq.svg',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'CHEQ',
				coinMinimalDenom: 'ncheq',
				coinDecimals: 9,
				coinGeckoId: 'cheqd-network',
				coinImageUrl: window.location.origin + '/public/assets/tokens/cheq.svg',
			},
		],
		gasPriceStep: {
			low: 25,
			average: 30,
			high: 50,
		},
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx'],
		explorerUrlToTx: 'https://cheqd.didx.co.za/transactions/{txHash}',
	}
```

</details>

<details>

<summary>Stargaze</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc.stargaze-apis.com',
		rest: 'https://rest.stargaze-apis.com',
		chainId: 'stargaze-1',
		chainName: 'Stargaze',
		stakeCurrency: {
			coinDenom: 'STARS',
			coinMinimalDenom: 'ustars',
			coinDecimals: 6,
			coinGeckoId: 'pool:ustars',
			coinImageUrl: window.location.origin + '/public/assets/tokens/stars.png',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('stars'),
		currencies: [
			{
				coinDenom: 'STARS',
				coinMinimalDenom: 'ustars',
				coinDecimals: 6,
				coinGeckoId: 'pool:ustars',
				coinImageUrl: window.location.origin + '/public/assets/tokens/stars.png',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'STARS',
				coinMinimalDenom: 'ustars',
				coinDecimals: 6,
				coinGeckoId: 'pool:ustars',
				coinImageUrl: window.location.origin + '/public/assets/tokens/stars.png',
			},
		],
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx'],
		explorerUrlToTx: 'https://www.mintscan.io/stargaze/txs/{txHash}',
	}
```

</details>

<details>

<summary>Chihuahua</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc.chihuahua.wtf',
		rest: 'https://api.chihuahua.wtf',
		chainId: 'chihuahua-1',
		chainName: 'Chihuahua',
		stakeCurrency: {
			coinDenom: 'HUAHUA',
			coinMinimalDenom: 'uhuahua',
			coinDecimals: 6,
			coinGeckoId: 'pool:uhuahua',
			coinImageUrl: window.location.origin + '/public/assets/tokens/huahua.png',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('chihuahua'),
		currencies: [
			{
				coinDenom: 'HUAHUA',
				coinMinimalDenom: 'uhuahua',
				coinDecimals: 6,
				coinGeckoId: 'pool:uhuahua',
				coinImageUrl: window.location.origin + '/public/assets/tokens/huahua.png',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'HUAHUA',
				coinMinimalDenom: 'uhuahua',
				coinDecimals: 6,
				coinGeckoId: 'pool:uhuahua',
				coinImageUrl: window.location.origin + '/public/assets/tokens/huahua.png',
			},
		],
		gasPriceStep: {
			low: 0.025,
			average: 0.03,
			high: 0.035,
		},
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx'],
		explorerUrlToTx: 'https://ping.pub/chihuahua/tx/{txHash}',
	}
```

</details>

<details>

<summary>Lum Network</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://node0.mainnet.lum.network/rpc',
		rest: 'https://node0.mainnet.lum.network/rest',
		chainId: 'lum-network-1',
		chainName: 'Lum Network',
		stakeCurrency: {
			coinDenom: 'LUM',
			coinMinimalDenom: 'ulum',
			coinDecimals: 6,
			coinGeckoId: 'pool:ulum',
			coinImageUrl: window.location.origin + '/public/assets/tokens/lum.svg',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('lum'),
		currencies: [
			{
				coinDenom: 'LUM',
				coinMinimalDenom: 'ulum',
				coinDecimals: 6,
				coinGeckoId: 'pool:ulum',
				coinImageUrl: window.location.origin + '/public/assets/tokens/lum.svg',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'LUM',
				coinMinimalDenom: 'ulum',
				coinDecimals: 6,
				coinGeckoId: 'pool:ulum',
				coinImageUrl: window.location.origin + '/public/assets/tokens/lum.svg',
			},
		],
		coinType: 118,
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx', 'ibc-go'],
		explorerUrlToTx: 'https://www.mintscan.io/lum/txs/{txHash}',
	}
```

</details>

<details>

<summary>Vidulum</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://mainnet-rpc.vidulum.app',
		rest: 'https://mainnet-lcd.vidulum.app',
		chainId: 'vidulum-1',
		chainName: 'Vidulum',
		stakeCurrency: {
			coinDenom: 'VDL',
			coinMinimalDenom: 'uvdl',
			coinDecimals: 6,
			coinGeckoId: 'vidulum',
			coinImageUrl: window.location.origin + '/public/assets/tokens/vdl.svg',
		},
		bip44: {
			coinType: 370,
		},
		bech32Config: Bech32Address.defaultBech32Config('vdl'),
		currencies: [
			{
				coinDenom: 'VDL',
				coinMinimalDenom: 'uvdl',
				coinDecimals: 6,
				coinGeckoId: 'vidulum',
				coinImageUrl: window.location.origin + '/public/assets/tokens/vdl.svg',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'VDL',
				coinMinimalDenom: 'uvdl',
				coinDecimals: 6,
				coinGeckoId: 'vidulum',
				coinImageUrl: window.location.origin + '/public/assets/tokens/vdl.svg',
			},
		],
		coinType: 370,
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx', 'ibc-go'],
		explorerUrlToTx: 'https://explorers.vidulum.app/vidulum/tx/{txHash}',
	}
```

</details>

<details>

<summary>Desmos</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc.mainnet.desmos.network',
		rest: 'https://api.mainnet.desmos.network',
		chainId: 'desmos-mainnet',
		chainName: 'Desmos',
		stakeCurrency: {
			coinDenom: 'DSM',
			coinMinimalDenom: 'udsm',
			coinDecimals: 6,
			coinGeckoId: 'pool:udsm',
			coinImageUrl: window.location.origin + '/public/assets/tokens/dsm.svg',
		},
		bip44: {
			coinType: 852,
		},
		bech32Config: Bech32Address.defaultBech32Config('desmos'),
		currencies: [
			{
				coinDenom: 'DSM',
				coinMinimalDenom: 'udsm',
				coinDecimals: 6,
				coinGeckoId: 'pool:udsm',
				coinImageUrl: window.location.origin + '/public/assets/tokens/dsm.svg',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'DSM',
				coinMinimalDenom: 'udsm',
				coinDecimals: 6,
				coinGeckoId: 'pool:udsm',
				coinImageUrl: window.location.origin + '/public/assets/tokens/dsm.svg',
			},
		],
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx', 'ibc-go'],
		explorerUrlToTx: 'https://explorer.desmos.network/transactions/{txHash}',
	}
```

</details>

<details>

<summary>Dig</summary>

```javascript
import { Bech32Address } from '@keplr-wallet/cosmos';

// https://github.com/osmosis-labs/osmosis-frontend/blob/master/src/config.ts#L531

export const EmbedChainInfos = 
	
	{
		rpc: 'https://rpc-1-dig.notional.ventures',
		rest: 'https://api-1-dig.notional.ventures',
		chainId: 'dig-1',
		chainName: 'Dig',
		stakeCurrency: {
			coinDenom: 'DIG',
			coinMinimalDenom: 'udig',
			coinDecimals: 6,
			coinGeckoId: 'pool:udig',
			coinImageUrl: window.location.origin + '/public/assets/tokens/dig.png',
		},
		bip44: {
			coinType: 118,
		},
		bech32Config: Bech32Address.defaultBech32Config('dig'),
		currencies: [
			{
				coinDenom: 'DIG',
				coinMinimalDenom: 'udig',
				coinDecimals: 6,
				coinGeckoId: 'pool:udig',
				coinImageUrl: window.location.origin + '/public/assets/tokens/dig.png',
			},
		],
		feeCurrencies: [
			{
				coinDenom: 'DIG',
				coinMinimalDenom: 'udig',
				coinDecimals: 6,
				coinGeckoId: 'pool:udig',
				coinImageUrl: window.location.origin + '/public/assets/tokens/dig.png',
			},
		],
		gasPriceStep: {
			low: 0.025,
			average: 0.03,
			high: 0.035,
		},
		features: ['stargate', 'ibc-transfer', 'no-legacy-stdTx', 'ibc-go'],
		explorerUrlToTx: 'https://ping.pub/dig/tx/{txHash}',
	}
```

</details>
