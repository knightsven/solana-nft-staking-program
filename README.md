# Solana NFT Staking

This is a reference NFT Staking Program & Client for Solana NFTs. This program is compatible with both Candy Machine v1 and Candy Machine v2 NFTs.

---

## Disclaimer

Use at your own risk. I will not answer any support-related inquiries about this repository. If you have any issues or want any changes made, please make a fork or open a pull request.

This program was initially built during Solana v1.8.0. A closed source version is actively maintained and updated for Shadowy Super Coders. I will not be sharing those updates.

Yes, I do know this program does not use Metaplex's new "Collections" feature. No, I will not implement it in this open source repository. Feel free to open a PR with the changes if you'd like.

---

## Support

If this repo helped you or your project in any way, any tips sent to A858S5BKJhrbc4mm8VcMMRWthBgtMu2AjDkAmrEKKEdD on Solana are highly appreciated.

# Staking Setup + Commands

## Reward token

- NOTE: Only have to do this in development. In production, the reward token should already exist and you just have to transfer some into the vault later.

`spl-token create-token --decimals 0`

- NOTE: Any decimal spl token will work. Just using 0 for development purposes.

`spl-token create-account <mint>`

- NOTE: replace <mint> with the returned mint address from above

`spl-token mint <mint> 1000000`

## Proof token

- NOTE: This should be a 0 decimal token. Only mint the number based on however large your NFT collection is. You can mint more if needed, but it shouldn't be necessary.

`spl-token create-token --decimals 0`

`spl-token create-account <mint>`

`spl-token mint <mint> 10000`

## Set admin, proof_mint and reward_mint variables in `program/src/lib.rs`

`cd program && cargo build-bpf`

- NOTE: If `cargo build-bpf` doesn't work for you, run `rm -rf ~/.cache/solana` and then re-run the build command again. This should force solana to re-download and link the bpf utilities.

## Deployment will cost about 3.31 sol

`solana program deploy /path/to/nft-staking-v2/program/target/deploy/staking.so`

## Set reward_mint, proof_mint and program_id variables in `client/src/main.rs`

## Run commands below in `client` directory

`cargo build`

## Generate vault and transfer reward+proof tokens into the vault

`cargo run -- generate_vault_address -e dev -s /path/to/deployer/id.json --min_period 60 --reward_period 10 --bonus_percentage 0 --bonus_period 30`

- NOTE: periods in seconds
- NOTE: You can use decimals for reward period, min_period, bonus_percentage, and bonus_period. If you do use decimals, do extensive testing to make sure you get the expected amount in rewards and the timing is all correct.

`spl-token transfer <reward_mint> 1000000 <vault-address> --fund-recipient`

- NOTE: second address is the vault address returned from `generate_vault_address` cmd

`spl-token transfer <proof_mint> 10000 <vault-address> --fund-recipient`

- NOTE: second address is the vault address returned from `generate_vault_address` cmd

## Add candy machine ID to whitelist

`cargo run -- add_to_whitelist -e dev -s /path/to/deployer/id.json --reward 1 --maxreward 30 --candy_machine <candy-machine-creator-address>`

- `<candy-machine-creator-address>` is the first creator address on the NFTs in your collection. This should be a creator with 0% share.

## Client commands

`cargo run -- stake -e dev -s /path/to/deployer/id.json --nft <nft-token-address>`

- Stakes your NFT into the program vault, gives you a proof token back

`cargo run -- stake_withdraw -e dev -s /path/to/deployer/id.json --nft <nft-token-address>`

- "Claims" your tokens on your nft without unstaking

`cargo run -- unstake -e dev -s /path/to/deployer/id.json --nft <nft-token-address>`

- Unstakes your NFT and claims tokens at the same time

- NOTE: Wait 60s before unstaking/claiming, as the min_period above is set to 60s. You can set this to 0 or 1 if you'd like users to instantly claim or unstake

`cargo run -- withdraw -e dev -s /path/to/deployer/id.json --amount 10`

- NOTE: This is an admin-only command that lets you withdraw your reward tokens out of stake vault. This is a failsafe in case you need to kill this staking program and create a new one.
