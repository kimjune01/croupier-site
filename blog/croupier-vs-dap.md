# Croupier vs DAP: Bearer Proofs vs Aggregate Privacy

*March 2026 | Croupier Blog*

The IETF's [Distributed Aggregation Protocol](https://datatracker.ietf.org/doc/draft-ietf-ppm-dap/) (DAP) is the most serious privacy-preserving measurement standard in development. Co-authored by engineers from Cloudflare and ISRG (the nonprofit behind Let's Encrypt), with production implementations from both organizations, DAP represents the cryptographic establishment's answer to "how do you measure things without seeing individual data?"

Croupier asks a different question. DAP asks "what are the aggregate statistics?" Croupier asks "did this specific channel cause this specific conversion?"

These are not competing answers to the same question. They are different questions with different architectures, different trust models, and different tradeoffs. Understanding where each one fits matters more than picking a winner.

## How DAP Works

DAP is defined in [`draft-ietf-ppm-dap-16`](https://datatracker.ietf.org/doc/html/draft-ietf-ppm-dap-16), currently on the IETF Standards Track. The cryptographic primitives live in a separate spec: [Verifiable Distributed Aggregation Functions](https://datatracker.ietf.org/doc/draft-irtf-cfrg-vdaf/) (VDAF, `draft-irtf-cfrg-vdaf-18`).

Four roles:

1. **Client** — holds an individual measurement. Calls the VDAF `shard()` function to split it into two shares, encrypts each share to a different Aggregator using HPKE, and uploads the report to the Leader.
2. **Leader Aggregator** — receives all client reports, decrypts its own share, forwards the Helper's encrypted share, and runs an interactive verification protocol with the Helper.
3. **Helper Aggregator** — operated by a *different organization* than the Leader. Decrypts its own share, participates in verification, computes its own aggregate share independently.
4. **Collector** — issues a query, receives both Aggregators' aggregate shares, calls `unshard()` to recover the final statistic. Never sees individual measurements.

The core guarantee: as long as at least one of the two Aggregators is honest, no individual measurement is ever seen in the clear. The VDAF's verification step also catches malformed inputs — a malicious client can't submit garbage that skews the aggregate.

Two concrete VDAFs ship with the spec. **Prio3** handles sums, means, histograms, and variance using additive secret sharing with zero-knowledge validity proofs. **Poplar1** solves the private heavy-hitters problem — finding frequently occurring strings without revealing any individual string.

## How Croupier Works

Croupier implements a different primitive: [blind signatures](https://en.wikipedia.org/wiki/Blind_signature), standardized in [Privacy Pass](https://www.rfc-editor.org/rfc/rfc9578.html) (RFC 9578).

Three roles:

1. **Advertiser (Issuer)** — generates a signing key, signs a batch of tokens (coupons), deposits the sealed batch with the relay.
2. **Publisher (Distributor)** — pulls the coupon book from the relay, embeds one token per ad impression as a URL query parameter.
3. **Customer (Bearer)** — carries the token from the ad to the conversion event. At conversion, presents the token.

The advertiser verifies their own signature at the conversion endpoint. Valid signature means the conversion came through the channel that distributed that coupon book. The relay (Croupier) passes sealed envelopes between advertisers and publishers. It never decrypts, never aggregates, never processes the tokens. A [structural test](https://github.com/kimjune01/croupier) verifies zero crypto imports exist in the relay's source code.

The guarantee: the relay cannot read what it relays (blind signatures), and no party can forge a token without the advertiser's private key.

## The Architectural Difference

DAP and Croupier solve different halves of the measurement problem.

**DAP computes aggregate statistics privately.** The Collector learns "Campaign A drove approximately 800 conversions across all publishers" but never sees which individual converted or which specific impression caused it. Privacy comes from aggregation plus optional differential privacy noise.

**Croupier produces individual attribution proofs.** The advertiser learns "this specific conversion presented a valid coupon from Publisher A's batch" but — due to blind signatures — can't link which specific token in the batch this was, preserving user privacy within the anonymity set of the batch. Privacy comes from unlinkability, not aggregation.

| | DAP | Croupier |
|---|---|---|
| Output | Aggregate statistic (sum, mean, histogram) | Per-conversion proof (valid signature or not) |
| Privacy mechanism | Secret sharing + aggregation + optional DP noise | Blind signatures (unlinkability within batch) |
| Trust model | Honest-minority: at least 1 of 2 Aggregators honest | Zero-trust on relay: can't read what it relays |
| Attribution logic | External (browser or MPC must match source → trigger) | Inherent (token is the join between impression and conversion) |
| Accuracy | Approximate (noise added for DP) | Exact (valid signature is binary) |
| Latency | Batched (reports must accumulate to `min_batch_size`) | Real-time (verify at conversion endpoint) |
| Cross-device | No (DAP alone) / Yes (IPA extension) | No (token must be carried from click to conversion) |
| Infrastructure | Two independent Aggregator operators + Collector | One relay (which could be self-hosted) |

## Trust Models Compared

DAP's trust model is **honest-minority**: privacy holds as long as at least one Aggregator executes the protocol honestly. If both Aggregators collude — or are compromised by the same attacker — they can reconstruct individual measurements. This requires two independent organizations to operate the Aggregators, which is both DAP's strength (cryptographic guarantee) and its operational cost (you need two trustworthy, independent operators).

ISRG operates [Janus](https://github.com/divviup/janus) as one Aggregator. Cloudflare operates [Daphne](https://github.com/cloudflare/daphne) as the other. Mozilla runs its own for Firefox deployments. The trust depends on these organizations not colluding.

Croupier's trust model is **zero-trust on the relay**. The relay handles HPKE-encrypted coupon books. Blind signatures mean it literally cannot read the tokens it passes. Even a fully compromised relay learns nothing about the coupons' contents. The advertiser doesn't trust the relay — they trust their own signing key.

The trust surfaces:

| What you trust | DAP | Croupier |
|---|---|---|
| Cryptographic primitives | Secret sharing, ZK proofs, HPKE | Blind signatures, HPKE |
| Operators | 2 independent Aggregators (at least 1 honest) | None (relay is blind) |
| Hardware | None (pure crypto) | None (pure crypto) |
| Browser vendor | Depends on deployment (Firefox PPA required browser) | No (token is a URL parameter) |
| Advertiser's own key | N/A | Yes (signing key security) |

Neither system requires trusting a TEE or hardware enclave. Both are pure cryptography. But DAP requires trusting the independence of two organizations, while Croupier requires the advertiser to secure their own private key.

## Where DAP Is Stronger

**Aggregate privacy guarantees.** DAP can enforce differential privacy at the aggregation layer — adding calibrated noise so that the aggregate statistic doesn't reveal whether any single individual's data was included. Croupier's per-conversion proofs are exact, which means an advertiser with a very small batch could potentially narrow down who converted. The batch size is the anonymity set, and the advertiser must choose batch sizes large enough to provide meaningful privacy.

**Resistance to Sybil attacks on the Collector.** Because DAP produces only aggregates, a malicious Collector can't learn individual conversions even with repeated queries (assuming proper DP budgeting). With Croupier, the advertiser sees each redemption individually — privacy for the user depends on the batch anonymity set, not on limiting the Collector's view.

**Standardization trajectory.** DAP is on the IETF Standards Track with multiple interoperable implementations. It has institutional backing from ISRG, Cloudflare, and Mozilla. The W3C's [Private Advertising Technology Community Group](https://www.w3.org/community/patcg/) (PATCG) is building attribution standards on top of DAP's aggregation layer.

**Cross-device potential.** The [IPA proposal](https://github.com/patcg-individual-drafts/ipa) (Interoperable Private Attribution), built by Mozilla and Meta, extends the MPC model to support cross-device attribution using encrypted match keys. A user who sees an ad on mobile and converts on desktop can be attributed without either Aggregator learning the user's identity. Croupier's bearer token model requires the same device to carry the token from impression to conversion.

## Where Croupier Is Stronger

**Per-channel attribution without noise.** An advertiser issuing 1,000 coupons to Publisher A and getting 80 back knows the conversion rate is exactly 8.0%, not "approximately 75-85 with ε=1.0 noise." For small advertisers with limited budgets, the noise floor in DAP-based systems can make the results unusable. DAP's `min_batch_size` requirement means you need enough reports before you get any result at all.

**No browser dependency.** DAP deployments to date have required browser integration. Firefox shipped [Privacy-Preserving Attribution](https://blog.mozilla.org/en/mozilla/privacy-preserving-attribution-for-advertising/) experimentally in Firefox 128 (July 2024), then removed it. Chrome uses its own TEE-based Attribution Reporting API instead of DAP. Safari has its own system (AdAttributionKit). The browser fragmentation means no DAP-based attribution standard works across all browsers today.

Croupier's token is a URL query parameter. It works in any browser, any app, any email client, any podcast player — anywhere that can pass a query string. No browser API required. No vendor adoption dependency.

**Advertiser-controlled measurement.** In DAP, the Collector receives an aggregate result computed by two Aggregators it doesn't operate. The Collector must trust that the Aggregators executed the protocol correctly. With Croupier, the advertiser verifies their own cryptographic signature at their own conversion endpoint. The measurement is self-sovereign.

**Operational simplicity.** DAP requires two independent organizations to operate Aggregators, maintain uptime, manage HPKE key rotation, and coordinate protocol execution. Croupier requires one relay that can be self-hosted, and the relay's correctness doesn't affect privacy because it can't read the tokens. A compromised or unreliable relay delays delivery but leaks nothing.

**Hijack resistance.** DAP aggregates measurements that are reported by the browser. If the browser's measurement is wrong — because a cookie was hijacked, a pixel was spoofed, or a bot triggered the event — DAP faithfully aggregates the wrong data. Garbage in, private garbage out.

Croupier's token is a signed proof that must be carried from the ad impression to the conversion event. A browser extension can't forge the advertiser's signature. A bot can't redeem a coupon at a real conversion endpoint (because bots don't make real purchases). The measurement is resistant to the input-level fraud that DAP inherits.

## The Attribution Gap

DAP is a generic aggregation protocol. It computes sums, means, and histograms over secret-shared inputs. It does not do attribution — matching an ad impression to a conversion event. That join has to happen somewhere else.

In Firefox's PPA experiment, the browser performed the attribution locally (on-device), then submitted the attributed result to DAP for aggregation. This means the browser decided which impression caused the conversion. If the browser is Chrome, the browser vendor is also the largest ad platform.

IPA moves the join into the MPC servers — both Aggregators cooperate to match source events (impressions) to trigger events (conversions) using doubly-blinded match keys. This is more flexible but requires "multiple orders of magnitude more" server-side computation than the TEE-based alternative, [per Google's own assessment](https://github.com/patcg-individual-drafts/ipa/issues/59).

With Croupier, the token *is* the join. The coupon travels from the impression to the conversion. No external system needs to match them. No browser needs to decide attribution. No MPC computation needs to correlate events. The physical movement of the token through the ad funnel is the attribution.

## When to Use Which

**Use DAP when:**
- You need aggregate statistics across large populations (telemetry, surveys, frequency estimation)
- Differential privacy guarantees are required by regulation or policy
- Cross-device attribution is necessary (via IPA extension)
- You're operating within a browser that supports the protocol

**Use Croupier when:**
- You need per-channel conversion rates without noise
- You're an advertiser who wants to verify attribution with your own keys
- Your ad channels span multiple platforms (web, app, email, podcast, print)
- You need hijack-resistant attribution (affiliate marketing, direct partnerships)
- You want to start measuring immediately without waiting for browser standards

**Use both when:**
- You want aggregate campaign-level statistics with formal privacy guarantees (DAP) calibrated against ground-truth per-channel conversion data (Croupier)
- You run some channels through browser-mediated ad networks (DAP/IPA territory) and others through direct publisher partnerships (Croupier territory)

## The Standards Question

DAP has institutional momentum. ISRG, Cloudflare, and Mozilla are investing in it. The IETF process gives it legitimacy. The PATCG is building on it.

Croupier builds on [Privacy Pass](https://www.rfc-editor.org/rfc/rfc9578.html) (RFC 9578) — an IETF standard that's already ratified, not a draft. Blind signatures have been proven secure since Chaum's 1983 paper. The token format is standardized. But the application to ad attribution is new, and there is no standards body coordinating its adoption for this use case.

DAP is a protocol looking for browser adoption. Croupier is a protocol that doesn't need it. Whether that's a strength or a limitation depends on whether you believe the browser should mediate ad measurement.

The [relay implementation](https://github.com/kimjune01/croupier) is 250 lines of infrastructure on top of standardized cryptographic primitives. DAP's implementations — Janus and Daphne — are substantially larger, reflecting the complexity of multi-party computation, interactive verification, and batched aggregation.

Complexity is not a criticism. DAP solves a harder problem (aggregate statistics with formal DP guarantees across a distributed system). Croupier solves a simpler problem (did this channel produce this conversion?) with simpler machinery. The right tool depends on which problem you have.

---

*Croupier is a blind relay for cryptographic coupon books. [Learn more](https://github.com/kimjune01/croupier) or [request early access](https://croupier.ad).*
