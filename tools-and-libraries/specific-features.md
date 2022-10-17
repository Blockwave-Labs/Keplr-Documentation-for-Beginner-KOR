# Specific features

Keplr CosmJs와 연결할 수 있었다면 [Use Keplr with CosmJs](cosmjs.md) 섹션으로 건너뛸 수 있습니다.

Keplr CosmJs에 쉽게 연결할 수 있는 방법을 지원하지만, 추가 기능을 제공하는 Keplr에 특화된 추가 기능이 있습니다.

### Using with Typescript <a href="#using-with-typescript" id="using-with-typescript"></a>

**`window.d.ts`**

```javascript
import { Window as KeplrWindow } from "@keplr-wallet/types";

declare global { 
    // eslint-disable-next-line @typescript-eslint/no-empty-interface 
    interface Window extends KeplrWindow {}
    }
```

`@keplr-wallet/types` 패키지에는 Keplr 관련된 유형 정의가 있습니다. TypeScript를 사용하는 경우 `npm install --save-dev @keplr-wallet/types` 또는 `yarn add -D @keplr-wallet/types`를 실행하여 `@keplr-wallet/types`를 설치합니다. 그런 다음 `@keplr-wallet/types` 창을 글로벌 창 개체에 추가하고 Keplr 관련 유형을 등록할 수 있습니다.

> @keplr-wallet/types 이외의 다른 패키지는 사용하지 않는 것이 좋습니다.
>
> * @keplr-wallet/type 이외의 다른 패키지는 현재 활발히 개발 중이며 이전 버전과의 호환성은 지원 범위에 포함되지 않습니다.
> * 현재 진행 중인 변경 사항이 있기 때문에 문서가 현재 최신 버전의 패키지로 업데이트되지 않습니다. 패키지가 안정되면 문서가 업데이트됩니다.

### Enable Connection <a href="#enable-connection" id="enable-connection"></a>

```
enable(chainIds: string | string[]): Promise<void>
```

`window.keplr.enable(chainIds)` 메서드는 확장이 현재 잠겨 있으면 잠금을 해제하도록 요청합니다. 사용자가 웹 페이지에 대한 권한을 부여하지 않은 경우 웹 페이지가 Keplr에 액세스할 수 있는 권한을 부여하도록 사용자에게 요청합니다.

`enable`메서드는 하나 이상의 체인 ID를 배열로 수신할 수 있습니다. chain-id 배열이 전달되면 아직 인증되지 않은 모든 체인에 대한 권한을 한 번에 요청할 수 있습니다.

사용자가 잠금 해제를 취소하거나 권한을 거부하면 오류가 발생합니다.

### Get Address / Public Key <a href="#get-address-public-key" id="get-address-public-key"></a>

```javascript
getKey(chainId: string): Promise<{
    // Name of the selected key store.
    name: string;
    algo: string;
    pubKey: Uint8Array;
    address: Uint8Array;
    bech32Address: string;
}>
```

웹 페이지에 권한이 있고 Keplr가 잠금 해제된 경우 이 함수는 다음 형식으로 주소와 공용 키를 반환합니다.

```javascript
{
    // Name of the selected key store.
    name: string;
    algo: string;
    pubKey: Uint8Array;
    address: Uint8Array;
    bech32Address: string;
    isNanoLedger: boolean;
}
```

또한 현재 선택한 키 저장소에 대한 닉네임을 반환하므로 웹 페이지가 사용자에게 선택한 현재 키 저장소를 보다 편리한 방식으로 표시할 수 있습니다.

반환 유형의 `isNanoLedger` 필드는 선택한 계정이 Ledger Nano에서 온 것인지 여부를 나타내는 데 사용됩니다. Ledger Nano의 현재 코스모스 앱은 direct (protobuf) 형식의 메시지들을 지원하지 않기 때문에, 이 필드는 amino 또는 direct 서명자를 선택하는 데 사용될 수 있습니다. [레퍼런스](https://docs.keplr.app/api/cosmjs.html#types-of-offline-signers)

### Sign Amino <a href="#sign-amino" id="sign-amino"></a>

```
signAmino(chainId: string, signer: string, signDoc: StdSignDoc): Promise<AminoSignResponse>
```

CosmJs `OfflineSigner`의 `signAmino` 와 비슷하지만 Keplr의 `signAmino`는 chain-id를 필수 매개 변수로 사용합니다. Amino-encoded `StdSignDoc`에 서명합니다.

### Sign Direct / Protobuf <a href="#sign-direct-protobuf" id="sign-direct-protobuf"></a>

```javascript
signDirect(chainId:string, signer:string, signDoc: {
    /** SignDoc bodyBytes */
    bodyBytes?: Uint8Array | null;

    /** SignDoc authInfoBytes */
    authInfoBytes?: Uint8Array | null;

    /** SignDoc chainId */
    chainId?: string | null;

    /** SignDoc accountNumber */
    accountNumber?: Long | null;
  }): Promise<DirectSignResponse>
```

CosmJs `OfflineDirectSigner`의 `signDirect` 와 비슷하지만 Keplr의 `signDirect`는 chain-id를 필수 매개 변수로 사용합니다. Proto-encoded `StdSignDoc`에 서명합니다.

### Request Transaction Broadcasting <a href="#request-transaction-broadcasting" id="request-transaction-broadcasting"></a>

```javascript
sendTx(
    chainId: string,
    tx: Uint8Array,
    mode: BroadcastMode
): Promise<Uint8Array>;
```

This function requests Keplr to delegates the broadcasting of the transaction to Keplr's LCD endpoints (rather than the webpage broadcasting the transaction). This method returns the transaction hash if it succeeds to broadcast, if else the method will throw an error. When Keplr broadcasts the transaction, Keplr will send the notification on the transaction's progress.

이 함수는 Keplr에게, 트랜잭션을 전하는 웹 페이지가 아닌 Keplr의 LCD 엔드포인트로 트랜잭션 전달을 위임하도록 요청합니다. 이 메서드는 트랜잭션 해시를 전 성공하면 반환하고 그렇지 않으면 오류를 반환합니다. Keplr 거래를 전하면, Keplr는 거래 진행 상황에 대한 통지를 보낼 것입니다.

### Request Signature for Arbitrary Message <a href="#request-signature-for-arbitrary-message" id="request-signature-for-arbitrary-message"></a>

```javascript
signArbitrary(
    chainId: string,
    signer: string,
    data: string | Uint8Array
): Promise<StdSignature>;
verifyArbitrary(
    chainId: string,
    signer: string,
    data: string | Uint8Array,
    signature: StdSignature
): Promise<boolean>;
```

이것은 [ADR-36](https://github.com/cosmos/cosmos-sdk/blob/main/docs/architecture/adr-036-arbitrary-signature.md)의 실험적인 구현입니. 이 기능은 사용자의 책임 하에 사용하세요.&#x20;

메인 용도는 `signArbitrary` API를 사용하여 ADR-36 서명을 요청하는 오프체인 계정의 소유권을 증명하는 것입니.&#x20;

Keplr `signArbitary` API를 사용하는 대신 ADR-36을 사용하는 `signAnimo` API로 sign doc을 요청하면 `signArbitary`로 작동합니다.

* Amino 형식의 sign doc만 지원합니다. (protobuf의 경우, 구현에 대한 [ADR-36](https://github.com/cosmos/cosmos-sdk/blob/main/docs/architecture/adr-036-arbitrary-signature.md) 요구사항이 완전히 지정되지 않았습니다)
* sign doc 메시지는 단일이고 메시지 유형은 "sign/MsgSignData"여야 합니다.
* sign doc "sign/MsgSignData" 메시지는 "signer"와 "data"를 값으로 지정해야 합니다. "data"는 base64로 인코딩되어야 합니다.
* sign doc의 chain\_id는 빈 문자열이어야 합니다("")
* sign doc의 memo는 빈 문자열이어야 합니다("")
* sign doc의 account\_number는 "0"이어야 합니다.
* sign doc의 sequence는 "0"이어야 합니다.
* sign doc의 fee는 `{gas: "0", amount: []}`이어야 합니다.

`signArbitrary` API를 사용할 때 `data` 매개 변수 유형이 `string`이면, 서명 페이지가 일반 텍스트로 표시됩니다.&#x20;

`verifyArbitrary`를 사용하면 `signArbitrary` API 또는 ADR-36 사양 표준으로 요청된 `signAmino` API에서 요청한 결과를 확인할 수 있습니다.

`verifyArbitrary`는 단순한 사용을 위해 구현되었습니다. `verifyArbitrary`는 현재 선택한 계정의 sign doc에 대한 확인 결과를 반환합니다. 계정이 현재 선택한 계정이 아닌 경우 오류가 발생합니다.&#x20;

`@keplr-wallet/cosmos`패키지 안에 있는 `verifyADR36Amino` 기능을 사용하거나 `verifyArbitrary` API를 사용하는 대신 사용자 자신의 구현을 사용하는 것이 좋습니다.

### Request Ethereum Signature <a href="#request-ethereum-signature" id="request-ethereum-signature"></a>

```
signEthereum(
  chainId: string,
  signer: string, // Bech32 address, not hex
  data: string | Uint8Array,
  type: 'message' | 'transaction'
)
```

This is an experimental implementation of native Ethereum signing in Keplr to be used by dApps on EVM-compatible chains such as Evmos.

It supports signing either [Personal Messages (opens new window)](https://eips.ethereum.org/EIPS/eip-191)or [Transactions (opens new window)](https://ethereum.org/en/developers/docs/transactions/), with plans to support [Typed Data (opens new window)](https://eips.ethereum.org/EIPS/eip-712)in the future.

이는 에브모스와 같은 EVM 호환 체인의 dApp에서 사용하기 위해 Keplr에서 네이티브 이더리움 서명을 실험적으로 구현한 것입니. 개[인 메시지 서명](https://eips.ethereum.org/EIPS/eip-191) 또는 트랜잭션 서명(새 창 열기)을 지원하며, 향후 유형 데이터(새 창 열기)를 지원할 계획입니다.

Notes on Usage:

* The `signer` field must be a Bech32 address, not an Ethereum hex address
* The data should be either stringified JSON (for transactions) or a string message (for messages). Byte arrays are accepted as alternatives for either.

### Interaction Options <a href="#interaction-options" id="interaction-options"></a>

```javascript
export interface KeplrIntereactionOptions {
  readonly sign?: KeplrSignOptions;
}

export interface KeplrSignOptions {
  readonly preferNoSetFee?: boolean;
  readonly preferNoSetMemo?: boolean;
}
```

Keplr v0.8.11+ offers additional options to customize interactions between the frontend website and Keplr extension.

If `preferNoSetFee` is set to true, Keplr will prioritize the frontend-suggested fee rather than overriding the tx fee setting of the signing page.

If `preferNoSetMemo` is set to true, Keplr will not override the memo and set fix memo as the front-end set memo.

You can set the values as follows:

```javascript
window.keplr.defaultOptions = {
    sign: {
        preferNoSetFee: true,
        preferNoSetMemo: true,
    }
}
```

## Custom event

### Change Key Store Event <a href="#change-key-store-event" id="change-key-store-event"></a>

```
keplr_keystorechange
```

When the user switches their key store/account after the webpage has received the information on the key store/account the key that the webpage is aware of may not match the selected key in Keplr which may cause issues in the interactions.

To prevent this from happening, when the key store/account is changed, Keplr emits a `keplr_keystorechange` event to the webpage's window. You can request the new key/account based on this event listener.

```
window.addEventListener("keplr_keystorechange", () => {
    console.log("Key store in Keplr is changed. You may need to refetch the account info.")
})
```
