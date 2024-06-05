# Superchain Presigned Pause

Status: DRAFT

Set the `SCRIPT` env var with the following command:

```
export SCRIPT=PresignPauseFromJson_eth_008
```

Then see [../../../PRESIGNED-PAUSE.md](../../../PRESIGNED-PAUSE.md) for the remainder of the
playbook.

## Additional Overrides

In order to enable the transaction simulation, prior to execution of the Guardian and Security
Council upgrades, this playbook requires additional overrides beyond what are typically included in
a presigned pause playbook.

The following is an explanation of those overrides. All other validation steps should be contained
in the PRESIGNED_PAUSE.md playbook linked to above.

### `0x09f7150D8c019BeF34450d6920f6B3608ceFdAf2` (1/1 Guardian Safe)

Links:
- [Etherscan](https://etherscan.io/address/0x09f7150D8c019BeF34450d6920f6B3608ceFdAf2).

The following two changes are both updates to the `modules` mapping, which is in [slot 1](https://github.com/safe-global/safe-contracts/blob/v1.3.0/contracts/examples/libraries/GnosisSafeStorage.sol#L10).

**Key:** `0x980c07ea7d4ff68ba3dc1784087a786aa4ab36b4fe0feb273e7b92f4944383de` <br/>
**Before:** `0x0000000000000000000000000000000000000000000000000000000000000000` <br/>
**After:** `0x0000000000000000000000000000000000000000000000000000000000000001` <br/>
**Meaning:** The `DeputyGuardianModule` at [`0x5dc91d01290af474ce21de14c17335a6dee4d2a8`](https://etherscan.io/address/0x5dc91d01290af474ce21de14c17335a6dee4d2a8) is now pointing to the sentinel module at `0x01`.
  This is `modules[0x5dc91d01290af474ce21de14c17335a6dee4d2a8]`, so the key can be
    derived from `cast index address 0x5dc91d01290af474ce21de14c17335a6dee4d2a8 1`.

**Key:** `0xcc69885fda6bcc1a4ace058b4a62bf5e179ea78fd58a1ccd71c22cc9b688792f` <br/>
**Before:** `0x0000000000000000000000000000000000000000000000000000000000000001` <br/>
**After:** `0x0000000000000000000000005dc91d01290af474ce21de14c17335a6dee4d2a8` <br/>
**Meaning:** The sentinel module (`address(0x01)`) is now pointing to the `DeputyGuardianModule` at [`0x5dc91d01290af474ce21de14c17335a6dee4d2a8`](https://etherscan.io/address/0x95703e0982140D16f8ebA6d158FccEde42f04a4C).
  This is `modules[0x1]`, so the key can be
    derived from `cast index address 0x0000000000000000000000000000000000000001 1`.

### `0x95703e0982140D16f8ebA6d158FccEde42f04a4C` (SuperchainConfig)

Links:
- [Etherscan](https://etherscan.io/address/0x95703e0982140D16f8ebA6d158FccEde42f04a4C).

- **Key:** 0xd30e835d3f35624761057ff5b27d558f97bd5be034621e62240e5c0b784abe68 <br/>
  **Value:** 0x00000000000000000000000009f7150d8c019bef34450d6920f6b3608cefdaf2 <br/>
  **Meaning:** The Guardian slot of the `SuperchainConfig` is set to the Guardian Safe address at `0x9f7150d8c019bef34450d6920f6b3608cefdaf2`.
     This is the same value it will be set to once [tasks/eth/010-1-guardian-upgrade](../010-1-guardian-upgrade/README.md) is executed. The slot can be computed as `cast keccak "superchainConfig.guardian"` then subtracting 1 from the result, as seen in the Superchain Config [here](https://github.com/ethereum-optimism/optimism/blob/op-contracts/v1.5.0-rc.1/packages/contracts-bedrock/src/L1/SuperchainConfig.sol#L23).


### `0x9BA6e03D8B90dE867373Db8cF1A58d2F7F006b3A` (Foundation Operations Safe)

The Safe will have the following overrides to set your address as the sole owner of the signing safe, to allow simulating the tx in Tenderly.

- **Key:** 0x0000000000000000000000000000000000000000000000000000000000000003 <br/>
  **Value:** 0x0000000000000000000000000000000000000000000000000000000000000001 <br/>
  **Meaning:** The number of owners is set to 1. The key can be validated by the location of the `ownerCount` variable in the [Safe's Storage Layout](https://github.com/safe-global/safe-smart-account/blob/v1.3.0/contracts/examples/libraries/GnosisSafeStorage.sol#L13).

- **Key:** 0x0000000000000000000000000000000000000000000000000000000000000004 <br/>
  **Value:** 0x0000000000000000000000000000000000000000000000000000000000000001 <br/>
  **Meaning:** The threshold is set to 1. The key can be validated by the location of the `threshold` variable in the [Safe's Storage Layout](https://github.com/safe-global/safe-smart-account/blob/v1.3.0/contracts/examples/libraries/GnosisSafeStorage.sol#L14).

The following two overrides are modifications to the [`owners` mapping](https://github.com/safe-global/safe-contracts/blob/v1.3.0/contracts/examples/libraries/GnosisSafeStorage.sol#L12). For the purpose of calculating the storage, note that this mapping is in slot `2`.
This mapping implements a linked list for iterating through the list of owners. Since we'll only have one owner (Multicall), and the `0x01` address is used as the first and last entry in the linked list, we will see the following overrides:
- `owners[1] -> 0xca11bde05977b3631167028862be2a173976ca11`
- `owners[0xca11bde05977b3631167028862be2a173976ca11] -> 1`

And we do indeed see these entries:

- **Key:** 0x316a0aac0d94f5824f0b66f5bbe94a8c360a17699a1d3a233aafcf7146e9f11c <br/>
  **Value:** 0x0000000000000000000000000000000000000000000000000000000000000001 <br/>
  **Meaning:** This is `owners[0xca11bde05977b3631167028862be2a173976ca11] -> 1`, so the key can be
    derived from `cast index address 0xca11bde05977b3631167028862be2a173976ca11 2`.

- **Key:** 0xe90b7bceb6e7df5418fb78d8ee546e97c83a08bbccc01a0644d599ccd2a7c2e0 <br/>
  **Value:** 0x000000000000000000000000ca11bde05977b3631167028862be2a173976ca11 <br/>
  **Meaning:** This is `owners[1] -> 0xca11bde05977b3631167028862be2a173976ca11`, so the key can be
    derived from `cast index address 0x0000000000000000000000000000000000000001 2`.