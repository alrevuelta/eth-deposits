# eth-deposits

Ethereum consensus layer is made of more than >500.000 validators, where each validator holds a pair of public/private keys and fulfills a set of duties in the network: proposing new blocks, attesting to new ones and participating in sync committees. However, not every single validator belongs to a completely uncorrelated entity. Some of the so-called **pools** or **operators**, or even solo stakers, run typically several hundreds of validators.

This repo aims to label the validators present in the beacon chain. In each `file.txt` you will find a set of public keys, one per line, containing all validators that a given entity controls or run. Moreover, in this document, you will find the methodology and sources that were used to make such classifications. Note that in some cases, the validators are publicly labeled by the entity that controls them, but in others, it's not, so we have to rely on different heuristics, theories and estimations to label them. This repo also includes some scripts to get them.

This repo contains three sources of information:
* Own
* ethsta.com
* beaconcha.in

**Disclaimer:**
Some of this data may be wrong or incomplete. If you think you can improve it, feel free to open a PR justifying why and your source of information. Use at your own risk. None of the contributors nor the creator of this repository is responsible of what you can do with this information.

# Tools

You may need the following tools in order to get the validators:
* synced endpoint of `chaind`
* a eth1 node (i.e. infura)
* a dune analytics account
* other external services or API, pool-specific

## coinbase
Coinbase follows a pattern when doing the deposits

```sql
SELECT 
  eth2."DepositContract_evt_DepositEvent".pubkey 
FROM 
  eth2."DepositContract_evt_DepositEvent" 
WHERE 
  eth2."DepositContract_evt_DepositEvent".evt_tx_hash in (
    SELECT 
      tx_hash 
    FROM 
      ethereum."traces" tr 
    WHERE 
      tr.block_number >= 11182202 
      AND tr."to" = '\x00000000219ab540356cBB839Cbe05303d7705Fa' 
      AND tr.success = TRUE 
      AND tr.FROM in (
        SELECT 
          ethereum.transactions.from 
        FROM 
          ethereum.transactions 
        where 
          ethereum.transactions.from in (
            SELECT 
              ethereum.transactions.from 
            FROM 
              ethereum.transactions 
            where 
              ethereum.transactions.to = '\x00000000219ab540356cBB839Cbe05303d7705Fa' 
            group by 
              ethereum.transactions.from 
            having 
              count(*) = 1
          ) 
          and ethereum.transactions.to = '\xA090e606E30bD747d4E6245a1517EbE430F0057e'
      )
  )
```

## lido

Lido is made of a set of 28 (and increasing) set of operators. They publicly recognize their validators.


Get all operators
```sql
SELECT f_operator_name FROM `high-hue-328212.chaind.t_operators`
group by f_operator_name
```

Get keys of operator `xxx`. Then Save the results, CSV (local file) and rename them to ".txt". Remove `f0_` at the beginning.

```sql
SELECT REPLACE(f_public_key, '\\x', '') FROM `high-hue-328212.chaind.t_operators`
where f_operator_name = 'xxx'
```

## bloxstaking

They sign with a graffitti. Iterate all proposed blocks and filter by graffitti.

```sql
SELECT 
  f_public_key 
FROM 
  `high-hue-328212.chaind.t_validators` 
where 
  f_index in (
    SELECT 
      DISTINCT b.f_proposer_index 
    FROM 
      high - hue - 328212.chaind.t_blocks b 
    WHERE 
      LOWER(
        FORMAT(
          "%t", 
          FROM_HEX(
            SUBSTR(b.f_graffiti, 3, 64)
          )
        )
      ) like '%bloxstaking.com%'
  )
```


## Allnodes

On top of the validators they run with Lido, they also run validators on their own.
```sql
SELECT 
  f_public_key 
FROM 
  `high-hue-328212.chaind.t_validators` 
where 
  f_index in (
    SELECT 
      DISTINCT b.f_proposer_index 
    FROM 
      high - hue - 328212.chaind.t_blocks b 
    WHERE 
      FORMAT(
        "%t", 
        FROM_HEX(
          SUBSTR(b.f_graffiti, 3, 64)
        )
      ) like '%Allnodes%'
  )

```

## dappnode

By default dappnode uses a graffiti that can be used to filter the validators that are running with that software. Note that the graffiti can be modified, so this is just an estimation. Not also that this is not labeling per se. A validator can be owned by a pool but also use dappnode at the sam time.

```sql
SELECT 
  f_public_key 
FROM 
  `high-hue-328212.chaind.t_validators` 
where 
  f_index in (
    SELECT 
      DISTINCT b.f_proposer_index 
    FROM 
      high - hue - 328212.chaind.t_blocks b 
    WHERE 
      LOWER(
        FORMAT(
          "%t", 
          FROM_HEX(
            SUBSTR(b.f_graffiti, 3, 64)
          )
        )
      ) like '%dappnode%')

```


## rocketpool

## kraken

* 0xa40dfee99e1c85dc97fdc594b16a460717838703
* 0x631c2d8d0d7a80824e602a79800a98d93e909a9e

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\xa40dfee99e1c85dc97fdc594b16a460717838703",
    "\\x631c2d8d0d7a80824e602a79800a98d93e909a9e"
)
```

## binance

* 0xBdD75A97c29294FF805FB2fEe65aBd99492b32A8
* 0x50fF765A993400CD62B61Cfa4Bb33B1dDF694eC7
* 0xf211Dbb151048f65895d99A446c5268198Af73D2
* 0x071A5f87Ca7cbDd86Ba03ab11f68C8fA2A542B91
* 0xfDE01891bC1DdA13Ad2B6027709777066290FD72
* 0xf3Dca7dc9265D92b17A054Cf43fE6e02c571553b
* 0x339c367c63d60aF280FB727140B7469675D478bc
* 0xF4fEae08C1Fa864B64024238E33Bfb4A3Ea7741d
* 0x996793790D726072273Ba5eEf1E15a032e847e2B
* 0x8ebcf6Dbe031f6ce1f3FDe97747f1B4Ad6ab88B2
* 0x32895327eEAbC019Bb3363A1cC14a078A3bb36fA
* 0xA2ABe03F3A906dc11e05e90489946E5844374708
* 0xe572d341Fdb292F0bf8964F3Ff9b0c2b9498f1C2
* 0xb53f8c05e52e3c49283528aa24f3494375325931
* 0x2f47a1c2db4a3b78cda44eade915c3b19107ddcc
* 0x82361cb56a94c803593fde178bab2f345178e901
* 0x0b75ef3fc3394548efed0d784afa32de81ad4923
* 0xeae401546b85ae9b8bf4b81b4fb5c4337f079c09
* 0x73483496eb9317ce4559f707c06b9377627a61b0
* 0x32c96d17d81615789357160b41da2ef8b712eba8
* 0xd1366d60d65bdfc92e5a5925fc4698a22e04e8c2
* 0x90ba5b3a3e9353f713bb873a44f2396429c1dd27
* 0xf17aced3c7a8daa29ebb90db8d1b6efd8c364a18

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\xBdD75A97c29294FF805FB2fEe65aBd99492b32A8",
    "\\x50fF765A993400CD62B61Cfa4Bb33B1dDF694eC7",
    "\\xf211Dbb151048f65895d99A446c5268198Af73D2",
    "\\x071A5f87Ca7cbDd86Ba03ab11f68C8fA2A542B91",
    "\\xfDE01891bC1DdA13Ad2B6027709777066290FD72",
    "\\xf3Dca7dc9265D92b17A054Cf43fE6e02c571553b",
    "\\x339c367c63d60aF280FB727140B7469675D478bc",
    "\\xF4fEae08C1Fa864B64024238E33Bfb4A3Ea7741d",
    "\\x996793790D726072273Ba5eEf1E15a032e847e2B",
    "\\x8ebcf6Dbe031f6ce1f3FDe97747f1B4Ad6ab88B2",
    "\\x32895327eEAbC019Bb3363A1cC14a078A3bb36fA",
    "\\xA2ABe03F3A906dc11e05e90489946E5844374708",
    "\\xe572d341Fdb292F0bf8964F3Ff9b0c2b9498f1C2",
    "\\xb53f8c05e52e3c49283528aa24f3494375325931",
    "\\x2f47a1c2db4a3b78cda44eade915c3b19107ddcc",
    "\\x82361cb56a94c803593fde178bab2f345178e901",
    "\\x0b75ef3fc3394548efed0d784afa32de81ad4923",
    "\\xeae401546b85ae9b8bf4b81b4fb5c4337f079c09",
    "\\x73483496eb9317ce4559f707c06b9377627a61b0",
    "\\x32c96d17d81615789357160b41da2ef8b712eba8",
    "\\xd1366d60d65bdfc92e5a5925fc4698a22e04e8c2",
    "\\x90ba5b3a3e9353f713bb873a44f2396429c1dd27",
    "\\xf17aced3c7a8daa29ebb90db8d1b6efd8c364a18"
)
```

## huobi

* 0x194bd70b59491ce1310ea0bceabdb6c23ac9d5b2
* 0xb73f4d4e99f65ec4b16b684e44f81aeca5ba2b7c

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x194bd70b59491ce1310ea0bceabdb6c23ac9d5b2",
    "\\xb73f4d4e99f65ec4b16b684e44f81aeca5ba2b7c"
)
```

## bitcoinsuisse

* 0xc2288b408dc872a1546f13e6ebfa9c94998316a2
* 0xdd9663bd979f1ab1bada85e1bc7d7f13cafe71f8
* 0x622de9bb9ff8907414785a633097db438f9a2d86
* 0x3837ea2279b8e5c260a78f5f4181b783bbe76a8b
* 0xec70e3c8afe212039c3f6a2df1c798003bf7cfe9
* 0x2a7077399b3e90f5392d55a1dc7046ad8d152348

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\xc2288b408dc872a1546f13e6ebfa9c94998316a2",
    "\\xdd9663bd979f1ab1bada85e1bc7d7f13cafe71f8",
    "\\x622de9bb9ff8907414785a633097db438f9a2d86",
    "\\x3837ea2279b8e5c260a78f5f4181b783bbe76a8b",
    "\\xec70e3c8afe212039c3f6a2df1c798003bf7cfe9",
    "\\x2a7077399b3e90f5392d55a1dc7046ad8d152348"
)
```

## stakedus

TODO

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x",
)
```

## stakefish

* 0x61c808d82a3ac53231750dadc13c777b59310bd9 (TODO: Unsure)
* 0x4c7052546a9e38a72b9731e334d8100b45e88cbc
* 0x1fa953ef0c089b342656194e569d2bc0de698add
* 0x4e017a192177abdae99250b6a0809dd76c856767
* 0xaa5104d696a3c8ab61331c87d9294c0ae1f5aa51
* 0x0194512e77d798E4871973d9cB9D7DDFC0fFd801

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    --"\\x61c808d82a3ac53231750dadc13c777b59310bd9",
    "\\x4c7052546a9e38a72b9731e334d8100b45e88cbc",
    "\\x1fa953ef0c089b342656194e569d2bc0de698add",
    "\\x4e017a192177abdae99250b6a0809dd76c856767",
    "\\xaa5104d696a3c8ab61331c87d9294c0ae1f5aa51",
    "\\x0194512e77d798E4871973d9cB9D7DDFC0fFd801"
)
```

## ankr

* 0x4069d8a3de3a72eca86ca5e0a4b94619085e7362

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x4069d8a3de3a72eca86ca5e0a4b94619085e7362"
)
```

## bitfinex

* 0x2b1df729083f6416861445d8aaac04ebdcd4a848
```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x2b1df729083f6416861445d8aaac04ebdcd4a848"
)
```

* 0x5a0036bcab4501e70f086c634e2958a8beae3a11
## okex

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x5a0036bcab4501e70f086c634e2958a8beae3a11",
)
```

## stakewise

Source: https://dune.com/jgf1/Stakewise

* 0x102f792028a56f13d6d99ed4ec8a6125de98582a"
* 0x5fc60576b92c5ce5c341c43e3b2866eb9e0cddd1"
* 0xAD4Eb63b9a2F1A4D241c92e2bBa78eEFc56ab990"

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x102f792028a56f13d6d99ed4ec8a6125de98582a",
    "\\x5fc60576b92c5ce5c341c43e3b2866eb9e0cddd1",
    "\\xAD4Eb63b9a2F1A4D241c92e2bBa78eEFc56ab990"
)
```

## piedao

* 0x66827bcd635f2bb1779d68c46aeb16541bca6ba8"

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x66827bcd635f2bb1779d68c46aeb16541bca6ba8"
)
```

## wexexchange

* 0xbb84d966c09264ce9a2104a4a20bb378369986db

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\xbb84d966c09264ce9a2104a4a20bb378369986db"
)
```

## kucoin

* 0xd6216fc19db775df9774a6e33526131da7d19a2c

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\xd6216fc19db775df9774a6e33526131da7d19a2c"
)
```

## poloniex

* 0x0038598ecb3b308ebc6c6e2c635bacaa3c5298a3

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x0038598ecb3b308ebc6c6e2c635bacaa3c5298a3"
)
```

## cream

* 0xb2c43455ee556dea95c0599b0d3f7d0abbf32fdb
* 0x197939c1ca20c2b506d6811d8b6cdb3394471074"

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\xb2c43455ee556dea95c0599b0d3f7d0abbf32fdb",
    "\\x197939c1ca20c2b506d6811d8b6cdb3394471074"
)
```

## bitpie:

* 0xa8582b5a0f615bc21d7780618557042be60b32ed

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x"
)
```

## 0x234ee9e35f

* 0x234ee9e35f8e9749a002fc42970d570db716453b

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x234ee9e35f8e9749a002fc42970d570db716453b"
)
```

## 0x711cd20bf6

* 0x711cd20bf6b436ced327a8c65a14491aa04c2ca1

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x711cd20bf6b436ced327a8c65a14491aa04c2ca1"
)
```

## 0xfb62633309

* 0xfb626333099a91ab677bcd5e9c71bc4dbe0238a8

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\xfb626333099a91ab677bcd5e9c71bc4dbe0238a8"
)
```

## 0x3b436fb33b

* 0x3b436fb33b79a3a754b0242a48a3b3aec1e35ad2

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x3b436fb33b79a3a754b0242a48a3b3aec1e35ad2"
)
```

## 0xa8582b5a0f

* 0xa8582b5a0f615bc21d7780618557042be60b32ed

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\xa8582b5a0f615bc21d7780618557042be60b32ed"
)
```

## 0xa3ae668b62

* 0xa3ae668b6239fa3eb1dc26daabb03f244d0259f0

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\xa3ae668b6239fa3eb1dc26daabb03f244d0259f0"
)
```

## 0x152c77c113

* 0x152c77c1131eb171b5547f2c4c33ff2c496984f1

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x152c77c1131eb171b5547f2c4c33ff2c496984f1"
)
```

## 0x38bc2db273

* 0x38bc2db2732a7efa43c89160118268ef0d15163d

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x38bc2db2732a7efa43c89160118268ef0d15163d"
)
```

## 0x0ec24e397a

* 0x0ec24e397a3592da41264209cc0582bc3bbc9776

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x0ec24e397a3592da41264209cc0582bc3bbc9776"
)
```

## 0x46d24de64a

* 0x46d24de64abe6a7ab6afda0389074ae8c7ed5d0f

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x46d24de64abe6a7ab6afda0389074ae8c7ed5d0f"
)
```

## 0x6aa35f8ae7

* 0x6aa35f8ae7c9aceb2a541b6576577fdcdae73100

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x6aa35f8ae7c9aceb2a541b6576577fdcdae73100"
)
```

## 0x27d71a464d

* 0x27d71a464da941118a4e056d0737274aa308a923

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x27d71a464da941118a4e056d0737274aa308a923"
)
```

## 0x51226113a3

* 0x51226113a35a29cda0030ef221e90baef221278b

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x51226113a35a29cda0030ef221e90baef221278b"
)
```

## 0xcb2a665406

* 0xcb2a66540680c344bab5f818d68c3e4b9d57363b

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\xcb2a66540680c344bab5f818d68c3e4b9d57363b"
)
```

## vitalik

* 0x1db3439a222c519ab44bb1144fc28167b4fa6ee6

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x1db3439a222c519ab44bb1144fc28167b4fa6ee6"
)
```

## lighthouse-team

* 0xfcd50905214325355a57ae9df084c5dd40d5d478

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\xfcd50905214325355a57ae9df084c5dd40d5d478"
)
```

## prysm-team

* 0x7badde47f41ceb2c0889091c8fc69e4d9059fb19

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x7badde47f41ceb2c0889091c8fc69e4d9059fb19"
)
```

## teku-team

* 0x43a0927a6361258e6cbaed415df275a412c543b5

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x43a0927a6361258e6cbaed415df275a412c543b5"
)
```

## nimbus-team

* 0x5efaefd5f8a42723bb095194c9202bf2b83d8eb6

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x5efaefd5f8a42723bb095194c9202bf2b83d8eb6"
)
```

## was.eth
* 0x010af29077a8520cc3f55be2031d7e37808ba137

```sql
SELECT f_validator_pubkey FROM `high-hue-328212.chaind.t_eth1_deposits`
where f_eth1_sender in
(
    "\\x010af29077a8520cc3f55be2031d7e37808ba137"
)
```


