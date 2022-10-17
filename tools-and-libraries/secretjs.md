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

SecretJs 링크: [https://www.npmjs.com/package/secretjs](https://www.npmjs.com/package/secretjs)&#x20;

SecretJs 사용 기본 사항은 CosmJs와 유사합니다. 자세한 내용은 [CosmJs](cosmjs.md) 섹션을 참조하세요.&#x20;

CosmJs와 SecretJs 사이의 한 가지 차이점은 Keplr의 `EnigmaUtils` 를 사용하는 것을 추천한다는 것입니. Keplr의 `EnigmaUtils` 를 사용하면 Keplr을 사용하여 암호화/복호화할 수 있으며, 해독된 트랜잭션 메시지는 사람이 읽을 수 있는 형식으로 사용자에게 표시됩니다.

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

웹 페이지는 SNIP-20 토큰을 Keplr의 토큰 목록에 추가할 수 있는 사용자 권한을 요청할 수 있습니다. 사용자가 요청을 거부할 경우 오류가 발생합니다. 동일한 계약 주소를 가진 SNIP-20이 이미 존재한다면, 아무 일도 일어나지 않을 것입니다.

#### Get SNIP-20 Viewing Key <a href="#get-snip-20-viewing-key" id="get-snip-20-viewing-key"></a>

```javascript
getSecret20ViewingKey(
    chainId: string,
    contractAddress: string
): Promise<string>;
```

Keplr에 등록된 SNIP-20 토큰의 viewing key를 반환합니다. 계약서 주소의 SNIP-20이 존재하지 않으면 오류를 발생시킵니다.

#### Interaction Options

SecretJs를 사용하는 경우에도 Keplr의 native API를 사용하여 상호 작용 옵션을 설정할 수 있습니다. 이 [섹션](specific-features.md#interaction-options)을 참조하십시오.\
