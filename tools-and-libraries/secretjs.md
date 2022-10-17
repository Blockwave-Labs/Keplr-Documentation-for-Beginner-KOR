# SecretJs

코스모스 생태계 안에 있는 Secret Network와 케플러를 연결하고 싶다면 SecretJs를 써서 쉽게 연결할 수 있습니다. SecretJs는 Secret Network 블록체인과 상호 작용하는 응용 프로그램을 작성하기 위한 자바스크립트 SDK입니. 만약 코스모스 생태계에 있는 Secret Network 에 대해서 궁금하다면 [이곳](https://scrt.network/about/about-secret-network/)에서 확인해볼 수 있습니다.

* TypeScript로 작성되었으며 유형 정의와 함께 제공됩니다.
* 핵심 데이터 구조에 대한 간단한 추상화를 제공합니다.
* 가능한 모든 메시지 및 트랜잭션 유형을 지원합니다.
* 가능한 모든 쿼리 유형을 표시합니다.
* 비밀 계약에 대한 입력/출력 암호화/복호화를 처리합니다.
* Node.js, 최신 웹 브라우저 및 React Native에서 작동합니다.

## Get started with SecretJs

### Connecting with SecretJs

SecretJS link: [https://www.npmjs.com/package/secretjs (opens new window)](https://www.npmjs.com/package/secretjs)The basics of using SecretJS is similar to CosmJS. Refer to the [Use with CosmJs](cosmjs.md) section for more information.

One difference between CosmJS and SecretJS is that we recommend using Keplr's `EnigmaUtils`. By using Keplr's `EnigmaUtils`, you can use Keplr to encrypt/decrypt, and the decrypted transaction messages are shown to the user in a human-readable format.

SecretJs 링크: [https://www.npmjs.com/package/secretjs](https://www.npmjs.com/package/secretjs)&#x20;

SecretJs 사용 기본 사항은 CosmJs와 유사합니다. 자세한 내용은 [CosmJs](cosmjs.md) 섹션을 참조하세요.&#x20;

CosmJs와 SecretJs 사이의 한 가지 차이점은 Keplr의 `EnigmaUtils` 를 사용하는 것을 추천한다는 것입니. Keplr의 `EnigmaUtils` 사를 사 용하면 Keplr을 사용하여 암호화/복호화할 수 있으며, 해독된 트랜잭션 메시지는 사람이 읽을 수 있는 형식으로 사용자에게 표시됩니다.

```javascript
// Enabling before using the Keplr is recommended.
// This method will ask the user whether or not to allow access if they haven't visited this website.
// Also, it will request user to unlock the wallet if the wallet is locked.
await window.keplr.enable(chainId);

const offlineSigner = window.getOfflineSigner(chainId);
**const enigmaUtils = window.getEnigmaUtils(chainId);**

// You can get the address/public keys by `getAccounts` method.
// It can return the array of address/public key.
// But, currently, Keplr extension manages only one address/public key pair.
// XXX: This line is needed to set the sender address for SigningCosmosClient.
const accounts = await offlineSigner.getAccounts();

// Initialize the gaia api with the offline signer that is injected by Keplr extension.
const cosmJS = new SigningCosmWasmClient(
    "https://lcd-secret.keplr.app/rest",
    accounts[0].address,
    offlineSigner,
    **enigmaUtils**
);
```

#### Suggest Adding SNIP-20 Tokens to Keplr <a href="#suggest-adding-snip-20-tokens-to-keplr" id="suggest-adding-snip-20-tokens-to-keplr"></a>

```javascript
async suggestToken(chainId: string, contractAddress: string): Promise<void>
```

The webpage can request the user permission to add a SNIP-20 token to Keplr's token list. Will throw an error if the user rejects the request. If a SNIP-20 with the same contract address already exists, nothing will happen.

#### Get SNIP-20 Viewing Key <a href="#get-snip-20-viewing-key" id="get-snip-20-viewing-key"></a>

```javascript
getSecret20ViewingKey(
    chainId: string,
    contractAddress: string
): Promise<string>;
```

Returns the viewing key of a SNIP-20 token registered in Keplr. If the SNIP-20 of the contract address doesn't exist, it will throw an error.

#### Interaction Options

You can use Keplr native API’s to set interaction options even when using SecretJS. Please refer to [this section](https://docs.keplr.app/api/#interaction-options).\
