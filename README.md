# Ferrum Network Bridge - Casper Smart Contracts

This repository contains the smart contracts used for the Bridge Pool on the Casper Network.


This contract has the following functionality:

- add liquidity to the pool
- remove liquidity from the pool
- withraw signed liquidity from the pool securely
- swap with another network
- get liquidity already in the pool
- add target for possible swap

## Table of Contents

1. [Getting Started](#getting-started)
2. [Prerequisites](#prerequisites)
3. [Makefile Commands](#makefile-commands)

4. [Usage](#usage)

5. [Installing and Interacting with the Contract using the Rust Casper Client](#installing-and-interacting-with-the-contract-using-the-rust-casper-client)

6. [Events](#events)

7. [Error Codes](#error-codes)

8. [Contributing](#contributing)

## Getting Started

To get started with using the smart contracts in this repository, you will need to have a working environment for Rust and Casper CLI.

```bash
cargo install casper-client
```
#### Notes: 
casper-client package cannot be installed inside the project root directory as a specific rustc version would be required.

## Prerequisites
You need to have a x86_64 CPU to build the code, as Casper dependencies for now don't support ARM (e.g. M1) architecture CPU's. It is recommended to use Linux, Debian-based distributions (e.g. Ubuntu 22.04).

First, you need to install Rust:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
To check for successful installation, one needs to run this command:
```bash
rustup --version
```

Secondly, you need to install CMake:
```bash
sudo apt-get -y install cmake
```
To check for successful installation, one needs to run this command:
```bash
cmake --version
```
To install casper-client, to interact with the contract using the CLI you must install these dependencies:
```bash
brew install pkg-config
brew install openssl
brew install libssl-dev
```
Since, Clang is also required for linux. Check if you have the required clang compiler already installed: 
```bash
clang --version 
```
To install clang on your linux distro:
```bash
sudo apt install clang
```

## Makefile Commands
For generating the cargo documentation: 
Change directory into contract directory by running

```bash
 cd contract
```

then run :

```bash
cargo doc
```

Make commands:
```bash
prepare:
	rustup target add wasm32-unknown-unknown
```
```bash
build-contract:
	cd contract && cargo build --release --target wasm32-unknown-unknown
	cd counter-call && cargo build --release --target wasm32-unknown-unknown
	cd erc20/erc20-token && cargo build --release --target wasm32-unknown-unknown

	wasm-strip contract/target/wasm32-unknown-unknown/release/bridge_pool.wasm 2>/dev/null | true
	wasm-strip counter-call/target/wasm32-unknown-unknown/release/counter-call.wasm 2>/dev/null | true
```
```bash
test-only:
	cd tests && cargo test
```
```bash
test: build-contract
	mkdir -p tests/wasm
	cp contract/target/wasm32-unknown-unknown/release/bridge_pool.wasm tests/wasm
	cp counter-call/target/wasm32-unknown-unknown/release/counter-call.wasm tests/wasm
	cp erc20/target/wasm32-unknown-unknown/release/erc20_token.wasm tests/wasm/erc20.wasm
	cd tests && cargo test
```
```bash	
clippy:
	cd contract && cargo clippy --all-targets -- -D warnings
```
```bash
check-lint: clippy
	cd contract && cargo fmt -- --check
	cd counter-call && cargo fmt -- --check
	cd tests && cargo fmt -- --check
```
```bash
lint: clippy
	cd contract && cargo fmt
	cd counter-call && cargo fmt
	cd tests && cargo fmt
```
```bash
clean:
	cd contract && cargo clean
	cd counter-call && cargo clean
	cd tests && cargo clean
	rm -rf tests/wasm
```

## Usage

### Set up the Rust toolchain

```bash
make prepare
```

### Compile smart contracts

```bash
make build-contract
```

### Run tests

```bash
make test
```
### Installing and Interacting with the Contract using the Rust Casper Client


##### Example deploy

The following is an example of deploying the installation of the contract via the Rust Casper command client.

```bash
casper-client put-deploy \
    --chain-name casper-test \
    --node-address http://44.208.234.65:7777 \
    --secret-key <path-to-key> \
    --session-path ./contract/target/wasm32-unknown-unknown/release/bridge_pool.wasm \
    --payment-amount 220000000000
```

##### Example add_liquidity
```bash
casper-client put-deploy \
    --chain-name casper-test \
    --node-address http://44.208.234.65:7777 \
    --secret-key <path-to-key> \
    --session-hash hash-2eaf3bf2cbc8e46f56ce04904592aa530141170fbee3473baeba4edfe9e87513 \
    --session-entry-point add_liquidity \
    --payment-amount 5000000000 \
    --session-arg "amount:u256='20'" \
    --session-arg "token_address:string='contract-package-wasm<token_address>'" \
    --session-arg "bridge_pool_contract_package_hash:string='contract-package-wasm<bridge_pool_contract_package_hash>'"
```

##### Example get_liquidity
```bash
casper-client put-deploy \
    --chain-name casper-test \
    --node-address http://44.208.234.65:7777 \
    --secret-key <path-to-key> \
    --session-hash hash-2eaf3bf2cbc8e46f56ce04904592aa530141170fbee3473baeba4edfe9e87513 \
    --session-entry-point get_liquidity \
    --payment-amount 5000000000 \
    --session-arg "token_address:string='contract-package-wasm<token_address>'"
```

##### Example remove_liquidity
```bash
casper-client put-deploy \
    --chain-name casper-test \
    --node-address http://44.208.234.65:7777 \
    --secret-key <path-to-key> \
    --session-hash hash-2eaf3bf2cbc8e46f56ce04904592aa530141170fbee3473baeba4edfe9e87513 \
    --session-entry-point remove_liquidity \
    --payment-amount 5000000000 \
    --session-arg "amount:u256='1'" \
    --session-arg "token_address:string='contract-package-wasm<token_address>'"
```

##### Example allow_target
```bash
casper-client put-deploy \
    --chain-name casper-test \
    --node-address http://44.208.234.65:7777 \
    --secret-key <path-to-key> \
    --session-hash hash-2eaf3bf2cbc8e46f56ce04904592aa530141170fbee3473baeba4edfe9e87513 \
    --session-entry-point allow_target \
    --payment-amount 5000000000 \
    --session-arg "token_address:string='contract-package-wasm<token_address>'" \
    --session-arg "token_name:string='<token_name>'" \
    --session-arg "target_network:u256='1'" \
    --session-arg "target_token:string='qwe'"
```

##### Example swap
```bash
casper-client put-deploy \
    --chain-name casper-test \
    --node-address http://44.208.234.65:7777 \
    --secret-key <path-to-key> \
    --session-hash hash-2eaf3bf2cbc8e46f56ce04904592aa530141170fbee3473baeba4edfe9e87513 \
    --session-entry-point swap \
    --payment-amount 5000000000 \
    --session-arg "amount:u256='1'" \
    --session-arg "target_network:u256='1'" \
    --session-arg "target_token:string='<target_token>'" \
    --session-arg "token_address:string='contract-package-wasm<token_address>'"
```

##### Example add_signer
```bash
casper-client put-deploy \
    --chain-name casper-test \
    --node-address http://44.208.234.65:7777 \
    --secret-key <path-to-key> \
    --session-hash hash-2eaf3bf2cbc8e46f56ce04904592aa530141170fbee3473baeba4edfe9e87513 \
    --session-entry-point add_signer \
    --payment-amount 5000000000 \
    --session-arg "signer:string='<signer>'"
```

##### Example remove_signer
```bash
casper-client put-deploy \
    --chain-name casper-test \
    --node-address http://44.208.234.65:7777 \
    --secret-key <path-to-key> \
    --session-hash hash-2eaf3bf2cbc8e46f56ce04904592aa530141170fbee3473baeba4edfe9e87513 \
    --session-entry-point add_signer \
    --payment-amount 5000000000 \
    --session-arg "signer:string='<signer>'"
```

##### Example withdraw_signed
```bash
casper-client put-deploy \
    --chain-name casper-test \
    --node-address http://44.208.234.65:7777 \
    --secret-key <path-to-key> \
    --session-hash hash-2eaf3bf2cbc8e46f56ce04904592aa530141170fbee3473baeba4edfe9e87513 \
    --session-entry-point withdraw_signed \
    --payment-amount 505000000000 \
    --session-arg "token_address:string='contract-package-wasm<token_address>'" \
    --session-arg "payee:string='<payee>'" \
    --session-arg "amount:u256='1'" \
    --session-arg "signature:string='<signature>'" \
    --session-arg "salt:string='<salt>'" \
    --session-arg "message_hash:string='<message_hash>'"
```


## Events

| Event name                | Included values and type                                                                                      |
| ------------------------- | ------------------------------------------------------------------------------------------------------------- |
| BridgeLiquidityAdded      | actor (Key) , token (Key), amount (U256)                                                                      |
| BridgeLiquidityRemoved    | actor (Key) , token (Key), amount (U256)                                                                      |
| BridgeSwap                | actor (Key) , token (Key), target_network: U256, target_token (String) , target_address (Key) , amount (U256) |
| TransferBySignature       | signer (Key), receiver (String), token (Key) , amount (U256)                                                  |


## Error Codes

| Code | Error                                               |
| ---- | --------------------------------------------------- |
| 1    | PermissionDenied                                    |
| 2    | WrongArguments                                      |
| 3    | NotRequiredStake                                    |
| 4    | BadTiming                                           |
| 5    | InvalidContext                                      |
| 6    | NegativeReward                                      |
| 7    | NegativeWithdrawableReward                          |
| 8    | NegativeAmount                                      |
| 9    | MissingContractPackageHash                          |
| 10   | InvalidContractPackageHash                          |
| 11   | InvalidContractHash                                 |
| 12   | WithdrawCheckErrorEarly                             |
| 13   | WithdrawCheckError                                  |
| 14   | NeitherAccountHashNorNeitherContractPackageHash     |
| 15   | UnexpectedContractHash                              |
| 16   | NotContractPackageHash                              |
| 17   | DictTargetTokenNotEqualTargetToken                  |
| 18   | NoTargetNetworkDictForThisToken                     |
| 19   | NoTargetTokenInAllowedTargetsDict                   |
| 20   | ClientDoesNotHaveAnyKindOfLiquidity                |
| 21   | ClientDoesNotHaveSpecificKindOfLiquidity           |
| 22   | AlreadyInThisTargetTokenDict                        |
| 23   | MessageAlreadyUsed                                  |
| 24   | NoValueInSignersDict                                |
| 25   | InvalidSigner                                       |
| 26   | CasperAccountHashParsing                            |
| 27   | WrongTokenName                                      |
| 28   | NoTokenInTokenContractPackageHashDict               |
| 29   | RecoverableSignatureTryFromFail                     |
| 30   | NonRecoverableSignatureTryFromFail                  |
| 31   | RecoverVerifyKeyFail                                |
| 32   | CheckedSubFail                                      |
| 33   | SaltHexFail                                         |
| 34   | SaltWrongSize                                       |
| 35   | SignatureHexFail                                    |
| 36   | NotBridgePoolContractPackageHash                    |
| 37   | EcdsaPublicKeyRecoveryFail                          |
| 38   | MessageHashHexDecodingFail                          |
| 39   | PublicKeyTryIntoFail                                |
| 40   | ImmediateCallerFail                                 |
| 41   | SignerWrongFormat                                   |

## Contributing

If you would like to contribute to this repository, please fork the repository and create a new branch for your changes. Once you have made your changes, submit a pull request and we will review your changes.

Please ensure that your code follows the style and conventions used in the existing codebase, and that it passes all tests before submitting a pull request.

## License

The smart contracts in this repository are licensed under the [MIT License](https://opensource.org/licenses/MIT).
