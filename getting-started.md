# Getting Started

초보자 친화적인 가이드를 제공합니다. 다음 단계에 따라 Keplr 구축을 시작해보세요!

## 1. Download Keplr

가장 먼저 개발 시스템에서 선택한 브라우저에 Keplr를 설치해주세요. [이곳](https://www.keplr.app/)에서 다운로드 하실 수 있습니다.

## 2. Detecting Keplr

`window.keplr` 를 확인하여 사용자 디바이스에 Keplr이 설치되어 있는지 여부를 확인할 수 있습니다. `window.keplr` 가 document.load 이후에 `undefined` 를 반환하는 경우, Keplr 설치되지 않습니다. 로드 이벤트가 상태를 확인할 때까지 기다리는 방법은 여러 가지가 있습니다. 아래 예를 참조해보세요.

`window.onload`에 함수를 등록할 수 있습니다.

```javascript
window.onload = async () => {
    if (!window.keplr) {
        alert("Please install keplr extension");
    } else {
        const chainId = "cosmoshub-4";
        
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
            "https://lcd-cosmoshub.keplr.app",
            accounts[0].address,
            offlineSigner,
        );
    }
} 
```

또는 문서 이벤트 수신기를 통해 문서의 준비 상태를 추적합니다.

```javascript
async getKeplr(): Promise<Keplr | undefined> {
    if (window.keplr) {
        return window.keplr;
    }
    
    if (document.readyState === "complete") {
        return window.keplr;
    }

    return new Promise((resolve) => {
        const documentStateChange = (event: Event) => {
            if (
                event.target &&
                (event.target as Document).readyState === "complete"
            ) {
                resolve(window.keplr);
                document.removeEventListener("readystatechange", documentStateChange);
            }
        };
        
        document.addEventListener("readystatechange", documentStateChange);
    });
}
```

동일한 결과를 얻을 수 있는 방법은 여러 가지가 있을 수 있으며 선호는 방법은 없습니다.

## Development Suite

기본적인 준비가 끝났네요! 이제 다양한 툴을 사용해서 본격적인 개발에 들어갈 수 있습니다. 자세한 내용은 Tools & Libraries 목차 내 각각의 카테고리를 참조해보세요.

### - CosmJs

케플러를 가장 쉽게 연동려면 [CosmJs ](tools-and-libraries/cosmjs.md)라이브러리를 활용해보세요.

### - Suggest Chain

Keplr 익스텐션에 통합 되어있지 않은 새로운 Cosmos SDK 기반 블록체인을 추가하도록 요청하려면, [Suggest Chain](tools-and-libraries/suggest-chain.md) 기능을 사용해보세요.&#x20;

### - Specific Features

Keplr를 CosmJs와 연결하는 방법을 알고 있다면 [CosmJs](tools-and-libraries/cosmjs.md) 섹션으로 건너뛸 수 있습니다. While Keplr supports an easy way to connect to CosmJs, there are additional functions specific to Keplr which provides additional features. Keplr는 CosmJs에 쉽게 연결할 수 있는 방법을 지원하지만, 추가 기능을 제공하는 Keplr에 특화된 추가 기능이 있습니다. 여기에서 특정 기능을 확인해보세요.

### - SecretJs

만약 secret-wasm 기능을 사용해야 한다면, [SecretJs](tools-and-libraries/secretjs.md)를 사용해보세요.&#x20;

