# bitcore-wallet

[![NPM Package](https://img.shields.io/npm/v/bitcore-wallet.svg?style=flat-square)](https://www.npmjs.org/package/bitcore-wallet)

A simple Command Line Interface Wallet using [Bitcore Wallet Service] (https://github.com/renuzit/bitcore-wallet-service-btcz) and its *official* client lib, bitcore-wallet-client-btcz] (https://github.com/renuzit/bitcore-wallet-client-btcz)

This can be used to operate Bitcoin and Bitcoin Cash wallets.


# Quick Guide

``` shell
# Use -h or BWS_HOST to setup the BWS URL (defaults to localhost:3001)
#
# Start a local BWS instance be doing:
# git clone https://github.com/bitpay/bitcore-wallet-service.git bws
# cd bws; npm install; npm start

cd bin

# Create a 2-of-2 wallet (~/.btczwallet.dat is the default filename where the wallet critical data will be stored)
#
# TIP: add -t for testnet, and -p to encrypt the credentials file
wallet create 'my wallet' 2-2
  * Secret to share:
    JevjEwaaxW6gdAZjqgWcimL525DR8zQsAXf4cscWDa8u1qKTN5eFGSFssuSvT1WySu4YYLYMUPT

# Check the status of your wallet
wallet status

  * Wallet my wallet [livenet]: 2-of-2 pending
    Missing copayers: 1

# Use -f or WALLET_FILE to setup the wallet data file

# Join the wallet as another copayer (add -p to encrypt credentials file)
wallet -f pete.dat join JevjEwaaxW6gdAZjqgWcimL525DR8zQsAXf4cscWDa8u1qKTN5eFGSFssuSvT1WySu4YYLYMUPT

export WALLET_FILE=pete.dat
wallet status

# Generate addresses to receive money
wallet address
  * New Address 3xxxxxx

# Check your balance
wallet balance

# Spend coins. Amount can be specified in btcz, bit or sat (default)
wallet send 1xxxxx 1000bit "1000 bits to mother"
  * Tx created: ID 01425517364314b9ac6017-e97d-46d5-a12a-9d4e5550abef [pending]
    RequiredSignatures: 2

# You can use 1000bit or 0.0001btcz or 100000sat. (Set BIT_UNIT to btcz/sat/bit to select output unit).

# List pending TX Proposals
wallet txproposals
  * TX Proposals:
    abef ["1000 bits to mother" by pete] 1,000 bit => 1xxxxx
      Missing signatures: 2

# Sign or reject TXs from other copayers
wallet -f pete.dat reject <id>
wallet -f pete.dat sign <id>

# List transaction history
wallet history
  a few minutes ago: => sent 1,000 bit ["1000 bits to mother" by pete] (1 confirmations)
  a day ago: <= received 1,400 bit (48 confirmations)
  a day ago: <= received 300 bit (62 confirmations)

# List all commands:
wallet --help


  ```


# Password protection

It is possible (and recommeded) to encrypt the wallet's credentials (.dat file). this is done
be adding the `-p` parameter to `join` or `create` or `genkey`. The password will be asked
interactively. Following commands that use the crendetials will require the password to work.

Password-based key derication function 2 (http://en.wikipedia.org/wiki/PBKDF2) is used to derive
the key to encrypt the data. AES is used to do the actual encryption, using the implementation
of SJCL (https://bitwiseshiftleft.github.io/sjcl/).


# Airgapped Operation

Air gapped (non connected) devices are supported. This setup can be useful if maximum security is needed, to prevent private keys from being compromised. In this setup, a device is installed without network access, and transactions are signed off-line. Transactions can be pulled from BWS using a `proxy` device, then downloaded to a pendrive to be moved to the air-gapped device, signed there, and then moved back the `proxy` device to be sent back to BWS. Note that Private keys are generated off-line in the airgapped device.


``` shell

# On the Air-gapped device

# Generate extended private key (add -t for testnet)
airgapped$ wallet genkey
  * Livenet Extended Private Key Created.

airgapped$ wallet export -o toproxy --nosign
  * Wallet data saved at toproxy without signing capability.


# On the proxy machine
proxy$ wallet import toproxy
  * Wallet Imported without signing capability.
proxy$ wallet join <secret>    # Or wallet create
proxy$ wallet address
proxy$ wallet balance

# It is not possible to sign transactions from the proxy device
proxy$ wallet sign
  [Error: You do not have the required keys to sign transactions]

# Export pending transaction to be signed offline
proxy$ wallet txproposals -o txproposals.dat

## Back to air-gapped device

# Sign them
airgapped$ wallet airsign txproposals.dat -o signatures.dat

# NOTE: To allow the airgapped device to check the transaction proposals being signed, the public keys of the copayers will be imported from the txproposals archive. That information is exported automatically by the proxy machine, and encrypted using copayer's xpriv derivatives.

## Back to proxy machine

# Send signatures to BWS
proxy$ wallet sign -i signatures.dat
  Transaction 014255.... signed by you.

```
