1. The oracle python app's source code is here: https://github.com/quex-tech/quex-v1-signer/tree/5a5f140fa00d6767ed55a3dbb1d101fd65155300

We build it into a docker container image via `./build.sh 0.0.6-cardano`.

2. We pack the container image into a Unified Kernel image via the https://github.com/quex-tech/quex-pack tool of the 0.0.11 version.

The command:

```
quex-pack -o signer-cardano.efi --kernel-cmdline "console=ttynull ip=10.13.192.143::10.13.192.1:255.255.192.0::eth0:off" docker-daemon:quex213/signer:0.0.6-cardano
```

The resulting EFI file can be downloaded here: https://drive.google.com/file/d/10AWS8Jklb_NvA9xrhAFTXVdcowyNB4C-/view?usp=drive_link

3. We verify the measurements of this TD via Quex online tool: https://verify.quex.tech/

Firmware image for the verification can be downloaded here: https://drive.google.com/file/d/1J0ptXOhluEB3Vo-sDA_dn6VYLP4jdGo2/view?usp=drive_link

The resulting measurements:

```
MRTD:  91eb2b44d141d4ece09f0c75c2c53d247a3c68edd7fafe8a3520c942a604a407de03ae6dc5f87f27428b2538873118b7
RTMR0: 5c09abbf7086ec0944a3f5da31c9b4af710d953e9ec51fcc0e43f6b1630323aa185099ab003bf223b882245bb195b6a5
RTMR1: 57429d9c9d2592028406dc0777c2a696cff0c4fdb5781a10eaf1fa537d2b40d021ed5e40b014e0fac7fc7fa865b8870f
RTMR2: 90a7d3562b05c8389f2a7ae8647aa77249a88f368bc58ac87f2599cd51b1962f730a0ee463f92002a859d863bd326cfc
RTMR3: 000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000 
```

4. The TD can reply to us with its public key and TD attestation quote.

The public key is `d2e0b6d3478c8c2e0e2e983967afe4eb1c9cac450c5e1fff209ea7ce8c030535b07137ef4d99c85863273b10f665f47c381d2b1e4b512195802be9bd9036eddc`.

The quote can be downloaded here: [quote.json](./quote.json).

We can confirm that the measurements are the same as in (3). Also, `reportdata` is equal to the public key.

5. We register the oracle on-chain via the `./oracles.py` script. The source code for Cardano off-chain scripts is here: https://github.com/quex-tech/cardano-oracle/tree/main/off-chain.

Transaction: https://preview.cardanoscan.io/transaction/be420346de90fd69b70ee47b6bbfaf4e9863fcde285aa8905ecf1bb21c021494

The resulting UTxO contains the oracle's public key.

It is marked with a custom asset. The full ID of the asset is the Oracle Pool ID, and its token name is the hash of the datum, which is guaranteed by the minting policy: https://github.com/quex-tech/cardano-oracle/blob/main/on-chain/src/SingleOraclePoolValidator.hs. Thus it's guaranteed, that the oracle pool always contains a single oracle with exactly this public key.

Pool ID is `b0344bbbe9caca56dabfcb583ce8d9038abdc80093badfc319d21b7fe3c170c711fc00aefbfeb9654203ac230181f05d60ecd8014cc61d74cb88af98`.

6. We make a request to the oracle for the data.

Request:

```
{
  "action": "2Hmf2Hmf2Hmf2HmAT2FwaS5iaW5hbmNlLmNvbVQvYXBpL3YzL3RpY2tlci9wcmljZYCf2HmfRnN5bWJvbEdBREFVU0RU//9A/9h5n0CAgEBYKjB4MDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMP9EdWludFgfLnByaWNlfHRvbnVtYmVyKjEwMDAwMDAwMHxmbG9vcv9fWED4ErNLhxwp6Jb/iIX2YSMl/ZRZwHnJ98r59Yv4qP+0zG180js4h/xrEpQ0PX37SVl9N21d43t2vIjkwLp9CV2XWECy9qGkz+k9UCGiGfSrup03FPSAtJZE9Ka2WEVkE+n/oh1XHQc3YRepr7hoiuynr/1HYp4jPNMUTtw0GEBa3yYl//8=",
  "relayer": "a4fed47f731213355aacea1f255d960b3c98967f40f0d1930234bfb1"
}
```

Action's hex is `d8799fd8799fd8799fd879804f6170692e62696e616e63652e636f6d542f6170692f76332f7469636b65722f7072696365809fd8799f4673796d626f6c4741444155534454ffff40ffd8799f40808040582a307830303030303030303030303030303030303030303030303030303030303030303030303030303030ff4475696e74581f2e70726963657c746f6e756d6265722a3130303030303030307c666c6f6f72ff5f5840f812b34b871c29e896ff8885f6612325fd9459c079c9f7caf9f58bf8a8ffb4cc6d7cd23b3887fc6b1294343d7dfb49597d376d5de37b76bc88e4c0ba7d095d975840b2f6a1a4cfe93d5021a219f4abba9d3714f480b49644f4a6b658456413e9ffa21d571d07376117a9afb8688aeca7affd47629e233cd3144edc3418405adf2625ffff` and can be inspected at https://cbor.nemo157.com/.

The request is `GET https://api.binance.com/api/v3/ticker/price?symbol=ADAUSDT`, the response is transformed via `.price|tonumber*100000000|floor` jq-filter and interpreted as a number.

Response:

```
{
  "msg": {
    "action_id": "sNeWqrQCNmMhoegyY2g+Ni/7UyPzILu0iiBh/pCkVSs=",
    "data_item": { "error": 0, "timestamp": 1766126808, "value": "GgImsJA=" },
    "relayer": "a4fed47f731213355aacea1f255d960b3c98967f40f0d1930234bfb1"
  },
  "sig": {
    "r": "KFHccHs9vhkqDgziPsUdTwM/E2MUVongaYcQ/4qdp7c=",
    "s": "caA2ADQ8LXDyYybwper+wVfaZz18wzM+dMSg2RNfVhg=",
    "v": 28
  }
}
```

Value's hex is `1a0226b090` which is CBOR for the number 36090000.

7. We can calculate PoolActionID as sha256(PoolID, ActionID), and compile https://github.com/quex-tech/cardano-oracle/blob/main/on-chain/src/ExampleUserValidator.hs with this value.

Pool Action ID is `361b4aff73c84d3dcaf6178ff36e0ae3e1833273a726fb9642bfde3a12f84f41`.

8. We lock funds at the demo validator.

9. We add the oracle response on-chain via https://github.com/quex-tech/cardano-oracle/blob/main/on-chain/src/OracleResponseValidator.hs. It checks the signature and mints an asset to mark the resulting datum.

Transaction: https://preview.cardanoscan.io/transaction/738ce1fc08f1fdbc5ba427c46ac392ac893918cc355ae8bddf62de72da5bf6af

Registered oracle UTxO is a reference input to this.

10. We spend the locked funds by adding the response UTxO as a reference input. The contract releases funds since ADA costs more than $0.25.

Transaction: https://preview.cardanoscan.io/transaction/51f1ae8c7e12a0a8df14002f87bdc5f8d9db1bbd7e0eb0da71df00511172371d