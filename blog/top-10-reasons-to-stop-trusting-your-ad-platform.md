# Top 10 Reasons to Stop Trusting Your Ad Platform
*March 2026 | Croupier Blog*

Ad platforms have a structural problem: the entity selling you impressions is also the entity reporting how well those impressions performed. That conflict doesn't require bad faith to produce bad data. It just requires misaligned incentives, and those are baked in.

Here are ten specific, documented reasons to treat platform-reported attribution as a starting point for investigation rather than a source of truth.

## 1. The Measured Party Controls the Measurement

Platforms self-report their own performance. There is no independent auditor verifying that a reported conversion actually happened, or that the platform caused it. When a platform tells you its ads produced 500 conversions this week, that number comes from the platform's own systems, optimized to make the platform look good. The measured party grading its own homework is not measurement.

## 2. Attribution Windows Are Set by the Platform

A view-through conversion credits the platform any time a user converts within a window after seeing an ad — even if they never clicked. The window length is set by the platform. A 30-day view-through window means the platform can claim credit for a purchase made a month after an impression that the user may not remember seeing. Platforms set these defaults to maximize reported credit.

## 3. Every Platform Counts the Same Conversion

Run Google, Meta, and TikTok ads simultaneously, then sum the attributed conversions from each dashboard. The total will exceed your actual sales by 30 to 50 percent or more. Each platform attributes conversions to itself using its own model. There is no cross-platform deduplication. The number on each dashboard is a claim, not a fact.

## 4. Advertisers Never See the Raw Log Data

Platforms do not share raw click and impression logs. Advertisers receive a processed dashboard — a claim derived from data they cannot inspect. You can see that the platform reports 1,200 clicks. You cannot see the IP addresses, timestamps, user agents, or behavioral signals behind those clicks. Without the raw log, there is no way to independently verify what you are being charged for.

## 5. Ad Verification Vendors Miss Most Bot Traffic

DoubleVerify and IAS are the two dominant third-party verification vendors. An Adalytics analysis found that DoubleVerify missed 21 percent of bot visits and IAS labeled confirmed bot traffic as valid human visitors 77 percent of the time. These were bots originating from known data center IP ranges — the easiest category to detect. The companies advertisers pay specifically to catch fraud are not catching it.

## 6. Browser Extensions Overwrite Attribution Cookies

The December 2024 Honey scandal revealed that a browser extension with tens of millions of users was systematically replacing affiliate tracking cookies at checkout. Any creator or publisher who had driven a sale lost attribution when Honey fired at the last second. Over 25 lawsuits were consolidated. The underlying mechanism — last-write-wins cookie attribution — is a property of the entire cookie-based system, not just Honey. Any extension with browser permissions can do this.

## 7. Forbes Ran a Secret Made-for-Advertising Subdomain

Adalytics documented that Forbes operated a hidden subdomain since 2017, serving real articles repurposed into ad-dense slideshow pages while bid requests misrepresented the inventory as coming from forbes.com. Roughly $770 million per quarter flows to made-for-advertising sites across the programmatic ecosystem. Premium brand adjacency is a label on the bid request. It is not a guarantee about where the ad actually ran.

## 8. Meta Has a Multi-Billion-Dollar Scam Ad Problem

Reuters' investigation found that a significant fraction of Meta's advertising revenue came from scam ads — illegal gambling, prohibited products, and fraudulent offers. Meta formed an anti-fraud team, reduced the problem materially, then dissolved the team. The platform that sells you impressions operates in the same auction as fraud operations. Inventory quality is not verified before you pay for it.

## 9. Google Was Found Guilty of Illegal Ad Monopoly Practices

In April 2025, a federal court found Google liable for illegally monopolizing the publisher ad server market and the ad exchange market. Google's own executives acknowledged that its ad exchange extracted outsized rents from the market. When a single company controls the buy side, the sell side, and the exchange in between, pricing and reporting are not set by competition. They are set by the monopolist.

## 10. You Have No Independent Verification of Any Conversion

The cumulative result of the above: every conversion in your dashboard is a number reported by a party with a financial interest in that number being high. Attribution windows, view-through credit, cross-platform double-counting, bot traffic, MFA inventory, cookie hijacking — each inflates the reported figure. None of it is verified by you.

The technical fix is straightforward. If the advertiser issues signed tokens before a campaign runs and checks its own signatures at the conversion endpoint, attribution becomes a cryptographic proof rather than a platform claim. Ed25519 signatures are small, fast, and do not require trusting any third party. The advertiser verifies its own proof. The platform's dashboard becomes a secondary signal, not the source of record.

---

*Croupier is a blind relay for cryptographic coupon books. [Learn more](https://github.com/kimjune01/croupier) or [request early access](https://croupier.ad).*
