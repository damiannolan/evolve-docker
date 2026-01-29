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

1. Create a `celestia-core.yaml` with a deployment plan. See the example below.
This defines the default hooks and ism.

```yaml
defaultHook:
  beneficiary: celestia1y3kf30y9zprqzr2g2gjjkw3wls0a35pfs3a58q
  oracleConfig:
    celestia:
      gasPrice: "200"
      tokenDecimals: 6
      tokenExchangeRate: "10000"
  oracleKey: celestia1y3kf30y9zprqzr2g2gjjkw3wls0a35pfs3a58q
  overhead:
    celestia: 50000
  owner: celestia1y3kf30y9zprqzr2g2gjjkw3wls0a35pfs3a58q
  type: interchainGasPaymaster
defaultIsm:
  type: testIsm
owner: celestia1y3kf30y9zprqzr2g2gjjkw3wls0a35pfs3a58q
requiredHook:
  type: merkleTreeHook
```

2. Run the following command to deploy the core infrastructure.

```bash
hyperlane core deploy --registry . --chain celestiadev --config configs/celestia-core.yaml --key.cosmosnative=6e30efb1d3ebd30d1ba08c8d5fc9b190e08394009dc1dd787a69e60c33288a8c
```

See the deployed contract addresses at `chains/{chainName}/addresses.yaml`.

3. Sync the core configuration with the deployment.

```bash
hyperlane core read --registry . --chain celestiadev --config configs/celestia-core.yaml
```

## Evolve Hyperlane core deployment

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