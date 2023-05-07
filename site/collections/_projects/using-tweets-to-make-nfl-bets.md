---
date: 2022-10-23 00:00:00
title: Public Cryptography with ECDSA Node
subtitle: Signature Basics
image: /uploads/public-key-cryptography.jpeg
---
You can run the react app [here](https://github.com/zakraicik/ecdsa-node).&nbsp;

In this project, we illustrate the basics of public key cryptography by building a simple react application that allows us to use a private key(never do this in practice) to look up user balances, and sign/execute transactions. The screenshot below shows the final product.&nbsp;

![](/uploads/final-app.png){: width="2232" height="696"}

## Public Key Cryptography

Public key cryptography, also known as asymmetric cryptography, is a cryptographic system that uses two separate keys for encryption and decryption processes: a public key and a private key. These keys are mathematically related but cannot be feasibly derived from one another. This system allows for secure communication and data exchange between parties without the need for sharing a common secret key.

The public key is openly shared and can be used by anyone to encrypt a message or verify a digital signature. The private key, on the other hand, is kept secret by its owner and is used to decrypt messages or create digital signatures.

Public key cryptography offers several benefits, including:

1. **Secure key exchange:** Participants can securely exchange keys over an insecure channel without the risk of eavesdropping, as the public key can be freely shared without compromising the private key.
2. **Digital signatures:** This allows for verifying the authenticity and integrity of a message, as the sender can sign the message using their private key, and the receiver can verify the signature using the sender's public key.
3. **Confidentiality:** Encrypted messages can only be decrypted by someone possessing the corresponding private key, ensuring that the message remains confidential.

## Elliptic Curve Digital Signature Algorithm

ECDSA, or Elliptic Curve Digital Signature Algorithm, is a widely-used digital signature algorithm based on elliptic curve cryptography (ECC) in public key cryptography. It is an adaptation of the Digital Signature Algorithm (DSA) that uses elliptic curve mathematics instead of traditional modular arithmetic. ECDSA provides a similar level of security as RSA and DSA but with smaller key sizes, resulting in faster computations and lower resource usage.

The main components of ECDSA are:

1. **Elliptic curve:** A mathematical curve defined by a specific equation. In ECDSA, the curve is chosen from a set of standardized curves that have been extensively studied and are considered secure.
2. **Key pair generation:** A private key is generated as a random number within a predefined range, and the corresponding public key is calculated by multiplying the private key with a predefined point (called the generator point) on the elliptic curve.
3. **Signing:** To sign a message, the sender generates a unique signature for the message using their private key and the elliptic curve mathematics. This signature ensures the authenticity and integrity of the message.
4. **Verification:** To verify the signature, the receiver uses the sender's public key and the elliptic curve mathematics to check whether the signature is valid. If the signature is valid, it confirms that the message was indeed sent by the sender and has not been tampered with.

ECDSA has become popular in various applications, including secure communications, cryptocurrencies (e.g., Bitcoin and Ethereum), and secure authentication. Its main advantage is the smaller key size compared to RSA, which allows for more efficient cryptographic operations while maintaining a similar level of security.

# Repository Design

---

This repository maintains both a client and a server. Each of these components is summarized below.&nbsp;

## Client

Technically, the client side is responsible for presenting information to the user, handling user interactions, and performing tasks on the user's device. In our repository, there are 3 core files to the client.&nbsp;

* `app.jsx`&nbsp;
* `wallet.jsx`&nbsp;
* `transfer.jsx`&nbsp;

Each of these is summarized below.&nbsp;

### `app.jsx`&nbsp;

This code defines a React functional component called `App`. The `App` component renders two child components, `Wallet` and `Transfer`, and passes down props to these components.

The `useState` hook from React is used three times to declare three state variables: `balance`, `address`, and `privateKey`, each initialized to an initial value of an empty string or zero. These state variables are used to manage the state of the application and are updated using the `setBalance`, `setAddress`, and `setPrivateKey` functions, respectively.

In the return statement, the `Wallet` component is rendered with props `balance`, `privateKey`, `setPrivateKey`, `setBalance`, `address`, and `setAddress`. The `Transfer` component is also rendered with props `setBalance`, `address`, and `privateKey`.

```javascript
import Wallet from "./Wallet";
import Transfer from "./Transfer";
import "./App.scss";
import { useState } from "react";

function App() {
  const [balance, setBalance] = useState(0);
  const [address, setAddress] = useState("");
  const [privateKey, setPrivateKey] = useState("");

  return (
    <div className="app">
      <Wallet
        balance={balance}
        privateKey={privateKey}
        setPrivateKey={setPrivateKey}
        setBalance={setBalance}
        address={address}
        setAddress={setAddress}
      />
      <Transfer setBalance={setBalance} address={address} privateKey={privateKey} />
    </div>
  );
}

export default App;
```

### `wallet.jsx`&nbsp;

This code defines a React functional component called `Wallet` that displays information about the user's wallet.

The `Wallet` component receives several props as arguments, including `address`, `setAddress`, `balance`, `setBalance`, `privateKey`, and `setPrivateKey`. These props are used to manage the state of the application and update the UI when the state changes.

In the `onChange` function, the `privateKey` state is updated with the value entered in the input field. Then, using the `ethereum-cryptography` library, the public key is generated from the private key and used to derive the user's Ethereum address. The `address` state is then updated with this value.

If the `address` state is not empty, the component sends a request to the server to get the balance associated with that address. If the request is successful, the `balance` state is updated with the value received from the server. If the address is empty, the `balance` state is set to zero.

Finally, the `Wallet` component renders a form with an input field for the user to enter their private key, and displays the user's address and balance in the UI.

```javascript
import server from "./server";
import * as secp from "ethereum-cryptography/secp256k1";
import { toHex } from "ethereum-cryptography/utils";

function Wallet({ address, setAddress, balance, setBalance, privateKey, setPrivateKey }) {
  async function onChange(evt) {
    const privateKey = evt.target.value;
    setPrivateKey(privateKey);
    const address = toHex(secp.getPublicKey(privateKey)).slice(-20);
    setAddress(address);
    if (address) {
      const {
        data: { balance },
      } = await server.get(`balance/${address}`);
      setBalance(balance);
    } else {
      setBalance(0);
    }
  }

  return (
    <div className="container wallet">
      <h1>Your Wallet</h1>

      <label>
        Private Key
        <input placeholder="Type in a private a key" value={privateKey} onChange={onChange}></input>
      </label>
      <div>
        Address: {address}
      </div>

      <div className="balance">Balance: {balance}</div>
    </div>
  );
}

export default Wallet;
```

### `transfer.jsx`&nbsp;

This code defines a React functional component called `Transfer` that allows users to send transactions.

The `Transfer` component receives several props as arguments, including `address`, `setBalance`, and `privateKey`. These props are used to manage the state of the application and update the UI when the state changes.

The component declares two state variables using the `useState` hook: `sendAmount` and `recipient`, both initialized to empty strings. These state variables are updated using the `setSendAmount` and `setRecipient` functions, respectively, when the user inputs data into the form.

The `transfer` function is called when the user submits the form. It first parses the `sendAmount` state as an integer and constructs a message by concatenating the user's address, the recipient's address, and the amount to be sent. The message is then hashed using the `ethereum-cryptography` library and the `secp256k1` algorithm is used to sign the message with the user's private key. The signature and recovery bit are included in the request body along with the sender's address, recipient's address, and the amount to be sent.

If the request is successful, the `balance` state is updated with the new balance returned from the server.If the request fails, an alert message is displayed with the error message returned from the server.

Finally, the `Transfer` component renders a form with input fields for the user to enter the amount and recipient address, and displays the user's private key in the UI. The form also includes a submit button that triggers the `transfer` function when clicked.

```javascript
import { useState } from "react";
import server from "./server";
import { sign } from "ethereum-cryptography/secp256k1";
import { keccak256 } from "ethereum-cryptography/keccak";
import { utf8ToBytes } from "ethereum-cryptography/utils";

function hashMessage(message) {
  const bytes = utf8ToBytes(message);
  return keccak256(bytes);
}

function Transfer({ address, setBalance, privateKey }) {
  const [sendAmount, setSendAmount] = useState("");
  const [recipient, setRecipient] = useState("");

  const setValue = (setter) => (evt) => setter(evt.target.value);

  async function transfer(evt) {
    evt.preventDefault();

    const amount = parseInt(sendAmount);
    const message = address + recipient + amount;
    const messageLength = message.length.toString();

    const formattedMessage = "\x19Ethereum Signed Message:\n" + messageLength + message;

    const messageHash = hashMessage(formattedMessage);
    const [signature, recoveryBit] = await sign(messageHash, privateKey, { recovered: true });

    try {
      const {
        data: { balance },
      } = await server.post(`send`, {
        sender: address, //public address
        amount,
        recipient,
        signature,
        recoveryBit
      });
      setBalance(balance);
    } catch (ex) {
      alert(ex.response.data.message);
    }
  }

  return (
    <form className="container transfer" onSubmit={transfer}>
      <h1>Send Transaction</h1>

      <label>
        Send Amount
        <input
          placeholder="1, 2, 3..."
          value={sendAmount}
          onChange={setValue(setSendAmount)}
        ></input>
      </label>

      <label>
        Recipient
        <input
          placeholder="Type an address, for example: 0x2"
          value={recipient}
          onChange={setValue(setRecipient)}
        ></input>
      </label>
      <div>
        Private Key: {privateKey}

      </div>

      <input type="submit" className="button" value="Transfer" />
    </form>
  );
}

export default Transfer;
```

## Server

In our simple illustration of public key cryptography, our server is governed by a single file `index.js`.The file first imports several Node.js packages, including `ethereum-cryptography`, `express`, and `cors`, which are used to handle cryptographic operations, build a web API, and enable cross-origin resource sharing (CORS) respectively.

The `hashMessage` function uses the `ethereum-cryptography` library to convert a message to bytes and hash the message using the keccak256 algorithm.

The server then creates an Express app and uses the `cors` and `express.json()` middleware to allow requests from different origins and parse incoming request bodies as JSON.

The `balances` object represents a list of addresses and their associated balances. The `/balance/:address` endpoint handles GET requests to retrieve the balance associated with a given address.

The `/send` endpoint handles POST requests to transfer funds from one address to another. It first retrieves the sender's and recipient's addresses, as well as the amount to be transferred, from the request body. The sender's and recipient's initial balances are set using the `setInitialBalance` function, which initializes a balance of 0 if the address has not been seen before.

The message is then constructed by concatenating the sender's and recipient's addresses with the amount to be transferred, and the message is formatted for signing. The message is then hashed and the signature and recovery bit provided by the client are used to recover the public key of the sender's private key. If the recovered public key does not match the expected sender's address, the transfer is considered invalid and an error is returned.

If the sender's balance is sufficient, the transfer is executed by updating the balances of the sender and recipient accordingly. Otherwise, an error is returned.

Finally, the server listens on port 3042 for incoming requests and logs a message to the console to indicate that it is running.

```javascript
const secp = require("ethereum-cryptography/secp256k1");
const { keccak256 } = require("ethereum-cryptography/keccak");
const { utf8ToBytes } = require("ethereum-cryptography/utils");
const { toHex } = require("ethereum-cryptography/utils");
const express = require("express");
const app = express();
const cors = require("cors");
const port = 3042;

function hashMessage(message) {
  const bytes = utf8ToBytes(message);
  return keccak256(bytes);
}

app.use(cors());
app.use(express.json());

const balances = {
  "8f2a62590609392a330f": 100, //c1acf26dea165249b9fd216563ada1c75870dc5c6400e2e32ebd970d4f543124
  "8773010fb85474d3cf21": 50, //36e2c0dc64e05535260b460165cc26b0230c9b27468920fe614fde00a162c026
  "ad9d83b0fde0ff6fca20": 75, //c335dd172aa8fc862847a9cd1f65729200941b9e1233740ff3f253c7fe66b1cf
};

app.get("/balance/:address", (req, res) => {
  const { address } = req.params;
  const balance = balances[address] || 0;
  res.send({ balance });
});

app.post("/send", (req, res) => {
  const { sender, recipient, amount, signature, recoveryBit } = req.body;

  setInitialBalance(sender);
  setInitialBalance(recipient);

  const message = sender + recipient + amount
  const messageLength = message.length.toString();

  const formattedMessage = "\x19Ethereum Signed Message:\n" + messageLength + message;

  const messageHash = hashMessage(formattedMessage);

  console.log("signature: ", signature);
  console.log(typeof signature);

  const signatureBuffer = Buffer.from(Object.values(signature));
  const recovered = secp.recoverPublicKey(messageHash, signatureBuffer, recoveryBit);

  if (toHex(recovered).slice(-20) != sender) {
    return res.status(400).send({ message: "Invalid signature" });
  }

  if (balances[sender] < amount) {
    res.status(400).send({ message: "Not enough funds!" });
  } else {
    balances[sender] -= amount;
    balances[recipient] += amount;
    res.send({ balance: balances[sender] });
  }
});

app.listen(port, () => {
  console.log(`Listening on port ${port}!`);
});

function setInitialBalance(address) {
  if (!balances[address]) {
    balances[address] = 0;
  }
}
```

# What We Learned

Although this is not a production grade project, it still demonstrates some key skills.&nbsp;

1. **Integration of public key cryptogrphy with a web application:** This project demonstrates how web applications can integrate public key crytography. This is a foundation concept for decentralized applications (dApps).
2. **Usage of React framework:** This project uses React, a popular JavaScript library for building user interfaces, to develop the web application's front-end. React allows for the efficient rendering and updating of the application's user interface based on changes to the application's state.
3. **Usage of `ethereum-cryptography` library:** This project uses the `ethereum-cryptography` library to perform cryptographic operations related to the Ethereum blockchain, such as generating public and private keys, signing and verifying messages, and hashing messages.
4. **Usage of Express framework:** This project uses Express, a popular Node.js framework for building web applications, to create an API that enables communication between the front-end and back-end of the application. Express allows for the creation of HTTP routes that handle incoming requests and return responses to the client.
5. **Implementation of wallet management features:** This project demonstrates the implementation of wallet management features, such as displaying the user's Ethereum address and balance, and enabling the transfer of funds between addresses.

Overall, this project showcases how a web application can use blockchain technology to build decentralized and secure applications, and how to leverage React and Express to build efficient and scalable applications.