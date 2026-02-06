# Hyperlane

Deploying Hyperlane core and warp routes.

## Add chain registry metadata

Running the following will auto-detect an EVM rpc running on `http://localhost:8545` and walk the user
through an interactive prompt to fulfill the chain details.

```bash
hyperlane registry init --registry .
```

For adding a cosmosnative protocol chain such as Celestia, its recommended to just copy the format used from 
the existing hyperlane-registry.

## Celestia Hyperlane core deployment

This guide details from-scratch deployment of Hyperlane core using the cosmosnative module.

As a prerequisite step export an env variable with the private key used for cosmosnative deployments.
```bash
export HYP_KEY_COSMOSNATIVE=6e30efb1d3ebd30d1ba08c8d5fc9b190e08394009dc1dd787a69e60c33288a8c
```

1. Create a `celestia-core.yaml` with a deployment plan. See the example below.
This defines the default hooks and ism.

```yaml
defaultHook:
  beneficiary: celestia1y3kf30y9zprqzr2g2gjjkw3wls0a35pfs3a58q
  oracleConfig:
    evolve:
      gasPrice: "1000000000"
      tokenDecimals: 18
      tokenExchangeRate: "1"
  oracleKey: celestia1y3kf30y9zprqzr2g2gjjkw3wls0a35pfs3a58q
  overhead:
    evolve: 300000
  owner: celestia1y3kf30y9zprqzr2g2gjjkw3wls0a35pfs3a58q
  type: interchainGasPaymaster
defaultIsm:
  domains:
    evolve:
      type: testIsm
  owner: celestia1y3kf30y9zprqzr2g2gjjkw3wls0a35pfs3a58q
  type: domainRoutingIsm
owner: celestia1y3kf30y9zprqzr2g2gjjkw3wls0a35pfs3a58q
requiredHook:
  type: merkleTreeHook
```

2. Run the following command to deploy the core infrastructure.

```bash
hyperlane core deploy --registry . --chain celestiadev --config configs/celestia-core.yaml
```

See the deployed contract addresses at `chains/{chainName}/addresses.yaml`.

3. Sync the core configuration with the deployment.

```bash
hyperlane core read --registry . --chain celestiadev --config configs/celestia-core.yaml
```

4. Updating the core configuration deployment.

The core configuration can be updated to, for example, add a new chain to the `DomainRoutingIsm` and `IGP`.
See the example diff below. Note this requires the new chain (e.g. `mockchain`) to have a chain metadata schema available in the registry.

```diff
defaultHook:
  address: "0x726f757465725f706f73745f6469737061746368000000040000000000000000"
  beneficiary: celestia1y3kf30y9zprqzr2g2gjjkw3wls0a35pfs3a58q
  oracleConfig:
    evolve:
      gasPrice: "1000000000"
      tokenDecimals: 18
      tokenExchangeRate: "1"
+    mockchain:
+      gasPrice: "100000000"
+      tokenDecimals: 6
+      tokenExchangeRate: "1"
  oracleKey: celestia1y3kf30y9zprqzr2g2gjjkw3wls0a35pfs3a58q
  overhead:
    evolve: 300000
+    mockchain: 300000
  owner: celestia1y3kf30y9zprqzr2g2gjjkw3wls0a35pfs3a58q
  type: interchainGasPaymaster
defaultIsm:
  address: "0x726f757465725f69736d00000000000000000000000000010000000000000001"
  domains:
    evolve:
      address: "0x726f757465725f69736d00000000000000000000000000000000000000000000"
      type: testIsm
+    mockchain:
+      type: testIsm
  owner: celestia1y3kf30y9zprqzr2g2gjjkw3wls0a35pfs3a58q
  type: domainRoutingIsm
owner: celestia1y3kf30y9zprqzr2g2gjjkw3wls0a35pfs3a58q
requiredHook:
  address: "0x726f757465725f706f73745f6469737061746368000000030000000000000001"
  type: merkleTreeHook
```

Apply the changes using the Hyperlane CLI and resync the configuration such that the addresses are reflected in the configuration file.

```bash
hyperlane core apply --registry . --chain celestiadev --config configs/celestia-core.yaml --key.cosmosnative=$PRIV_KEY

hyperlane core read --registry . --chain celestiadev --config configs/celestia-core.yaml
```

## EVM Hyperlane core deployment

Ensure the `HYP_KEY` env variable is set with a valid private key for a funded account.

1. Run the following command to generate a deployment configuration.

```bash
hyperlane core init --registry . --advanced --config configs/evolve-core.yaml
```

Choose the deployment options. Recommended for testnet/devnet deployments.
- Ism: testIsm
- DefaultHook: protocolFee (optionally IGP but protocolFee requires less configuration)
- RequiredHook: merkleTreeHook

2. Run the following command to deploy the core contracts. Select the chain from the interactive prompt.

```bash
hyperlane core deploy --registry . --config configs/evolve-core.yaml
```

See the deployed contract addresses at `chains/{chainName}/addresses.yaml`.

3. Sync the core configuration with the deployment.

```bash
hyperlane core read --registry . --chain evolve --config configs/evolve-core.yaml
```

## Warp routes

Warp Routes are managed in the Hyperlane Registry under `./deployments/warp_route/[SYMBOL]/`. Every warp route consists of two files: `[name]-config.yml` and `deploy-config.yml`.

## Deploy a new Warp Route

1. Create a `[name]-deploy.yaml` file in the correct folder in the registry. Use an existing file as a template. You can remove the `interchainSecurityModule` entry if the default ISM should be used.

2. Run `hyperlane warp apply --wd [name]-deploy.yaml` to create a new warp route. Deployment requires a private key with funds for each included chain. Ownership will be transferred to the specified owner address at the end. You can initially use the deployer key as the owner, then change the owner entry and run Warp apply again once everything works correctly (see next section).