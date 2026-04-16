# TESLA-for-5G: Tamarin Formal Verification

Tamarin Prover model for **TF5** (TESLA-for-5G), a broadcast authentication scheme for 5G System Information. The model is based on the TESLA proof of Cremers [1], and is extended with a GG09 identity-based signature bootstrap and TESLA chain renewal. All 19 lemmas verify.

## Paper

> **TESLA-for-5G: Broadcast Authentication for 5G Networks Using TESLA**
> Under submission.

## Dependencies

- [Tamarin Prover](https://tamarin-prover.com/) 1.12.0 or later

## Commands

```bash
# Full proof suite
tamarin-prover --prove --stop-on-trace=SEQDFS tf5.spthy
```

On an Intel Core Ultra 7 155U (14 cores, 32 GB RAM), the full suite completes in 18.87 s.
With Tamarin 1.12.0, the default DFS search can stall on `renewed_chain_is_executable`; `SEQDFS` verifies the full suite.

## Lemmas

| Lemma | Type | Purpose |
|---|---|---|
| `loop_origin` | reuse, induction | Every chain key descends from a generated root |
| `loopchain` | reuse, induction | Chain link relation is a well-formed hash chain |
| `chain_keys_unique` | reuse, induction | Each chain position binds to a unique key |
| `knows_only_expired_chain_keys` | reuse, induction | Adversary learns chain keys only after disclosure |
| `knows` | reuse, induction | Adversary's knowledge is bounded by compromise |
| `bootstrap_is_executable` | exists-trace | Sanity: IBS bootstrap path is reachable |
| `bootstrap_parameter_agreement` | all-traces | UE and gNB agree on TESLA parameters at bootstrap |
| `bootstrap_freshness` | all-traces | Stale bootstrap messages with expired signatures are rejected |
| `stale_gnb_signing_key_rejected` | all-traces | Bootstraps using expired gNB signing keys are rejected |
| `bootstrap_compromise_bounded` | all-traces | IBS compromise does not retroactively forge bootstraps |
| `tesla_is_executable` | exists-trace | Honest run completes end-to-end with SIB1 timing respected |
| `base_chain_is_executable` | exists-trace | Bootstrap + SIB1 on base chain reachable without renewal |
| `compromise_enables_forgery` | exists-trace | Sanity: compromise still enables forgery even when SIB1 timing is respected |
| `authentic` | all-traces | Accepted SIB1 was sent by an honest gNB unless the cell is compromised |
| `sib1_compromise_isolation` | all-traces | SIB1 key compromise does not forge past SIB1s in other cells |
| `renewal_is_executable` | exists-trace | Honest chain renewal is reachable without compromise |
| `renewed_chain_is_executable` | exists-trace | SIB1 verification on a renewed chain is reachable without compromise |
| `renewal_without_rebootstrap` | exists-trace | Renewal is driven by an earlier verified SIB1 unless the cell is already compromised |
| `renewal_commitment_authenticity` | all-traces | Renewed chain commitment is authentic to the gNB |

## Abstractions & Limitations

This model verifies the core TF5 protocol but abstracts several features from the paper:

- **Symbolic time only** — Tamarin cannot model wall-clock time. The TESLA safe-packet test, signing-key validity period, and signature freshness checks are all enforced as restrictions over symbolic trace ordering of verification, disclosure, and expiration events.
- Idealized cryptography — GG09 IBS, hash, and HMAC are idealized as black-box primitives.
- Chain renewal covers the same-parameter path (`sp_next = 0`) only; the new-parameter path (`sp_next = 1`) is not modeled.


## Acknowledgements

The authors used generative AI tools to assist with model development and documentation drafting. All lemmas, proofs, and claims in this artifact were reviewed and verified by the authors.

## References

[1] Cas Cremers, Charlie Jacomme, and Philip Lukert. *Subterm-Based Proof Techniques for Improving the Automation and Scope of Security Protocol Analysis.* In 2023 IEEE 36th Computer Security Foundations Symposium (CSF), pp. 200–213, 2023. [doi:10.1109/CSF57540.2023.00001](https://doi.org/10.1109/CSF57540.2023.00001)
