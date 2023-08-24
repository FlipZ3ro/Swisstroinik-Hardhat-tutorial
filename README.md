# SWISSTRONIK HARDHAT DEPLOY TUTORIAL

## ALAT :

[VSCODE](https://code.visualstudio.com/download)

[NODEJS](https://nodejs.org/dist/v18.17.1/node-v18.17.1-x64.msi
)

[FAUCET](https://faucet.testnet.swisstronik.com/)


## Referensi

[DOKUMEN RESMI](https://swisstronik.gitbook.io/swisstronik-docs/build-on-swisstronik/contract-deployment-hardhat)

[DISCORD](https://discord.gg/c5WBWaSYkt)


### Langkah 1: Persiapan dan Instalasi Hardhat

Pastikan Anda sudah memiliki Node.js dan npm (Node Package Manager) diinstal pada sistem Anda sebelum melanjutkan dengan panduan ini. Jika belum, Anda dapat mengunduhnya dari situs resmi Node.js.
Langkah pertama adalah menginstal Hardhat sebagai dev dependency pada proyek Anda. Buka terminal dan jalankan perintah berikut:

```
npm install --save-dev hardhat
```
Setelah instalasi selesai, Anda dapat membuat proyek Hardhat dengan perintah:

```
npx hardhat
```
Pilih opsi "Create a JavaScript project" dan tekan enter.

Pastikan Anda menginstal hardhat toolbox dengan menjalankan
```
npm install --save-dev @nomicfoundation/hardhat-toolbox
```

### Langkah 2: Konfigurasi hardhat.config.js

Setelah berhasil membuat proyek Hardhat, langkah selanjutnya adalah mengkonfigurasi berkas hardhat.config.js. Ini adalah berkas konfigurasi utama di mana Anda dapat menyesuaikan jaringan, plugin, pengaturan kompilasi, dan lain-lain.
Buka berkas hardhat.config.js dan pastikan konfigurasinya seperti ini:

```
require("@nomicfoundation/hardhat-toolbox");

module.exports = {
  solidity: "0.8.19",
  networks: {
    swisstronik: {
      url: "https://json-rpc.testnet.swisstronik.com/",
      accounts: ["0xd5..."], // Ganti dengan kunci pribadi Anda yang diawali dengan "0x"
    },
  },
};
```

### Langkah 3: Menulis dan Kompilasi Smart Contract

Buatlah direktori "contracts" di dalam direktori proyek Anda dan buka berkas .sol untuk menulis smart contract. Beri nama berkas ini misalnya Hello_swtr.sol.
Salin kode smart contract berikut ke dalam berkas Hello_swtr.sol:

```
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.19;

contract Swisstronik {
    string private message;

    constructor(string memory _message) payable {
        message = _message;
    }

    function setMessage(string memory _message) public {
        message = _message;
    }

    function getMessage() public view returns(string memory) {
        return message;
    }
}
```
Lalu, untuk mengompilasi smart contract, jalankan perintah berikut di terminal:

```
npx hardhat compile
```

### Langkah 4: Deploy Smart Contract

Buatlah berkas deploy.js di dalam direktori "scripts" dan salin kode berikut:

```
const hre = require("hardhat");

async function main() {
  const contract = await hre.ethers.deployContract("Swisstronik", ["Hello Swisstronik!!"]);

  await contract.waitForDeployment();

  console.log(`Kontrak Swisstronik dideploy ke alamat: ${contract.address}`);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```
Untuk mendeploy kontrak, jalankan perintah berikut di terminal:

```
npx hardhat run scripts/deploy.js --network swisstronik
```
Setelah berhasil, Anda akan melihat pesan "Kontrak Swisstronik dideploy ke alamat: ..." di terminal.

### Langkah 5: Interaksi dengan Kontrak - Transaksi

Pastikan Anda telah menginstal @swisstronik/swisstronik.js dengan menjalankan perintah:

```
npm install --save @swisstronik/swisstronik.js
```
Buat berkas setMessage.js di dalam direktori "scripts" dan salin kode berikut:

masukan sc yang sudah di buat tadi di bagian SC

```
const hre = require("hardhat");
const { encryptDataField, decryptNodeResponse } = require("@swisstronik/swisstronik.js");

const sendShieldedTransaction = async (signer, destination, data, value) => {
  const rpclink = hre.network.config.url;
  const [encryptedData] = await encryptDataField(rpclink, data);
  return await signer.sendTransaction({
    from: signer.address,
    to: destination,
    data: encryptedData,
    value,
  });
};

async function main() {
  const contractAddress = "SC";
  const [signer] = await hre.ethers.getSigners();
  const contractFactory = await hre.ethers.getContractFactory("Swisstronik");
  const contract = contractFactory.attach(contractAddress);
  const functionName = "setMessage";
  const messageToSet = "Hello Swisstronik!!";
  const setMessageTx = await sendShieldedTransaction(signer, contractAddress, contract.interface.encodeFunctionData(functionName, [messageToSet]), 0);
  await setMessageTx.wait();
  console.log("Tanda Terima Transaksi: ", setMessageTx);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

Untuk menjalankan skrip interaksi ini, gunakan perintah berikut di terminal:

```
npx hardhat run scripts/setMessage.js --network swisstronik
```

### Langkah 6: Interaksi dengan Kontrak - Panggilan

Buat berkas getMessage.js di dalam direktori "scripts" dan salin kode berikut:

masukan sc yang sudah di buat tadi di bagian SC

```
const hre = require("hardhat");
const { encryptDataField, decryptNodeResponse } = require("@swisstronik/swisstronik.js");

const sendShieldedQuery = async (provider, destination, data) => {
  const rpclink = hre.network.config.url;
  const [encryptedData, usedEncryptedKey] = await encryptDataField(rpclink, data);
  const response = await provider.call({
    to: destination,
    data: encryptedData,
  });
  return await decryptNodeResponse(rpclink, response, usedEncryptedKey);
};

async function main() {
  const contractAddress = "SC";
  const [signer] = await hre.ethers.getSigners();
  const contractFactory = await hre.ethers.getContractFactory("Swisstronik");
  const contract = contractFactory.attach(contractAddress);
  const functionName = "getMessage";
  const responseMessage = await sendShieldedQuery(signer.provider, contractAddress, contract.interface.encodeFunctionData(functionName));
  console.log("Pesan yang Didekripsi:", contract.interface.decodeFunctionResult(functionName, responseMessage)[0]);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```
Jalankan perintah berikut untuk menjalankan skrip interaksi panggilan:

```
npx hardhat run scripts/getMessage.js --network swisstronik
```
Dengan menyelesaikan langkah-langkah di atas, Anda telah mengikuti tutorial untuk mengatur lingkungan Hardhat, mengkonfigurasi berkas, menulis, mengompilasi, mendeploy, dan berinteraksi dengan kontrak pintar di jaringan Swisstronik menggunakan skrip transaksi dan panggilan.
