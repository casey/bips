```
  BIP: ?
  Layer: Consensus (soft fork)
  Title: P2Q: SegWit version 3 spending rules
  Authors: Casey Rodarmor <casey@rodarmor.com>
  Status: Draft
  Type: Specification
  Assigned: ?
  License: BSD-3-Clause
  Requires: 341, 342
```

## Abstract

This document defines P2Q (Pay-to-Quantum-safe), a new SegWit version 3
output type with spending rules identical to SegWit version 1
([BIP 341][BIP341]/[BIP 342][BIP342]). P2Q exists so that a future soft fork
can disable key-path spending for version 3 outputs only, providing opt-in
quantum resistance without affecting existing P2TR outputs.

## Copyright

This document is licensed under the 3-clause BSD license.

## Motivation

Taproot ([BIP 341][BIP341]) exposes the tweaked public key directly in the
scriptPubKey. If a cryptographically relevant quantum computer breaks ECDLP,
an attacker can derive the private key from any exposed public key and spend
via key-path. [BIP 360][BIP360] describes this threat model in detail.

Users can already use a nothing-up-my-sleeve (NUMS) internal key in P2TR to
make key-path spending practically impossible, but there is no
consensus-enforced guarantee that key-path is disabled. A distinct SegWit
version enables clean consensus-level disabling of key-path spending via a
future soft fork.

P2Q requires no new validation logic today. If quantum computers never
threaten ECDLP, P2Q outputs continue working exactly as taproot. If they do,
a future soft fork can make single-element witness stacks (key-path spends)
invalid for version 3 outputs, forcing all spends through script-path. Full
quantum resistance would additionally require a quantum-safe signature scheme
or other spending mechanism available via script-path. The details of that
future soft fork are out of scope for this proposal.

## Specification

A P2Q output is a native SegWit output ([BIP 141][BIP141]) with version 3 and
a 32-byte witness program. The scriptPubKey is 34 bytes:

    OP_3 OP_PUSHBYTES_32 <32-byte-tweaked-public-key>

All spending rules are identical to those defined in [BIP 341][BIP341] for
version 1 outputs with 32-byte witness programs, with version 3 substituted
for version 1. This includes key-path spending, script-path spending, and
annex handling. Tapscript validation (leaf version 0xc0) follows
[BIP 342][BIP342].

P2Q addresses use bech32m encoding ([BIP 350][BIP350]) with SegWit version 3,
giving mainnet addresses the prefix `bc1r`.

## Rationale

A new SegWit version is the cleanest mechanism for creating a class of outputs
whose spending rules can be tightened by a future soft fork. It requires no
new opcodes, no changes to the signature hashing algorithm, and no changes to
the control block format.

[BIP 360][BIP360] (P2MR, SegWit version 2) provides immediate quantum
resistance by removing key-path spending entirely. P2Q takes a different
approach: it preserves full taproot functionality today and defers key-path
disabling to a future soft fork, if and when quantum computers pose a real
threat.

Using a NUMS internal key in P2TR does not provide equivalent protection.
A quantum computer capable of breaking ECDLP could compute the discrete log
of any curve point, including those believed to have no known discrete log,
making key-path spendable for any P2TR output.

Version 3 is chosen because version 2 is allocated to P2MR
([BIP 360][BIP360]).

## Backward Compatibility

P2Q is a soft fork: non-upgraded nodes treat version 3 SegWit outputs as
anyone-can-spend. Non-upgraded wallets can send to P2Q addresses if they
support bech32m ([BIP 350][BIP350]).

[BIP141]: bip-0141.mediawiki
[BIP341]: bip-0341.mediawiki
[BIP342]: bip-0342.mediawiki
[BIP350]: bip-0350.mediawiki
[BIP360]: bip-0360.mediawiki
