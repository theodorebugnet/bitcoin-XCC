Bitcoin Core - forked for XCC
=====================================

This is a fork of Bitcoin Core which supports signing transactions locked with
P2WSH scripts used in XCLAIM Commit. The main use for it is for invoking the
`signrawtransactionwithkey` RPC: there is no wallet support or integration.

There is no explicit support for partially-signed transactions, and it has not
yet been properly tested in a context where each party signs separately.

XCC
---
See the thesis (link TBA).

Script output that this fork can sign:
```
<vault_sig> OP_CHECKSIGVERIFY <user_sig> OP_CHECKSIG OP_IFDUP OP_NOTIF
    <checkpoint_CSV_locktime> OP_CHECKSEQUENCEVERIFY
OP_ENDIF
```

This can be spent by a double vault + user multisig signing, or by the vault
alone once the CSV locktime has passed.

This is the output of Issue and Checkpoint transactions. Checkpoint, Recover,
Escape and Redeem transactions must spend from this output. Escape and Redeem
use conventional single-sign outputs (e.g. P2WPK, but this is not specified,
could be P2PKH, P2SH-P2WPK, etc., subject to preference).

The Recover transaction uses an HTLC script in its output. Spending from that
is currently not supported by this fork.

Building
-----
Follow the instructions in the Bitcoin documentation. Consider disabling
unnecessary components during the configure step if this is not going to be used
as the primary node, e.g.:
```
./configure --without-miniupnpc --disable-bench --without-gui
```

Usage
---
When spending from an Issue or Checkpoint transaction, use the
`signrawtransactionwithkey` RPC using this fork to generate a valid witness. The
signed transaction can then be broadcast on a separate node (e.g. using mainline
Bitcoin Core), at will.

E.g.

./bitcoin-cli signrawtransactionwithkey <prepared spending TX hex> '["private key(s)"]' '[{"txid": "<issue or checkpoint txid>", "vout":<vout>, "scriptPubKey":"<issue/checkpoint scriptPubKey (P2WSH)>", "witnessScript":"21<vaultpubkey(33 bytes)>ad21<clientpubkey(33 bytes)>ac7364032a0040b268", "amount":<amount spent from the checkpoint>}]'


Full Node
---
Due to being a fork of Bitcoin Core, it can also be used as a full node, especially if it's compiled with support for wallet, gui etc. This may be convenient for testing and development.

Obviously, being an unofficial fork, it is not recommended for use on the mainnet.
