# 7 Signs Your Ad Attribution Is Lying to You
*February 2026 | Croupier Blog*

Attribution data has a structural problem: the parties being measured are also the ones doing the measuring. Every ad platform reports conversions using its own attribution model, optimized to make itself look good. The result is numbers that diverge from reality in predictable ways. Here are seven concrete symptoms to look for.

## 1. Platform-Reported Conversions Sum to More Than Actual Orders

Export attributed conversions from every platform you run — Meta, Google, TikTok, programmatic — and add them up. If the total exceeds your actual order count, multiple platforms are claiming credit for the same events. This is the most direct evidence that attribution is inflated.

This isn't a rounding error. Platform self-reporting routinely inflates conversions by 30-50% because each platform applies its own attribution window to the same conversion pool. A customer who clicked a Google ad three days ago and a Meta ad yesterday may show up as a conversion in both dashboards. The platforms aren't lying about individual clicks — they're each applying their own logic to claim the win.

## 2. A Retargeting or Brand Search Channel Claims Most of the Credit

Last-click attribution systematically over-credits the channel closest to checkout. Retargeting and branded search tend to capture users who had already decided to convert — they intercept demand, they don't create it. When these channels dominate your attributed conversions, the number tells you where users ended up, not what drove the decision.

The Honey scandal made this structural problem visible at scale. PayPal's browser extension overwrote affiliate cookies at checkout, replacing the cookie belonging to the creator who drove the sale with Honey's own identifier. Last-write-wins is how last-click models work; Honey was simply explicit about exploiting it. Any channel with late-funnel touchpoints has the same opportunity to absorb credit from channels that did the actual work.

## 3. Conversions Appear With No Matching Order in Your Backend

Pull attributed conversions from a platform for a given day and cross-reference them against orders in your own system. If you find conversions with no corresponding order ID, the platform counted something your system doesn't recognize as a sale.

This happens through broad match between events: micro-conversions (add-to-cart, page visits) counted as conversions if the pixel fires on the wrong page, or outright bot-generated conversion events that pass the platform's fraud filter. In the Adalytics analysis of DoubleVerify and IAS, ad verification vendors missed between 21% and 77% of confirmed bot traffic — declared bots from known data center IPs, not sophisticated adversaries. If the vendors paid to catch fraud are missing that much, platform-side fraud filters aren't catching it all either.

## 4. Your ROAS Looks Suspiciously Round or Consistent

Real purchase behavior is noisy. If your reported return on ad spend is consistently 4.0x or 3.8x across weeks with different products, different audiences, and different seasonality, that consistency is itself a signal. Attribution models that smooth over variance tend to be filling gaps with modeled data rather than measured events.

Google's Performance Max, for instance, shows you impression counts per placement across its network but does not break out clicks, cost, or conversions by placement. Google's own description is that placement reports are a "brand safety tool, not a performance report." If you can't see where your spend actually went, the ROAS figure is being computed over a black box. A round, stable number from a black box deserves scrutiny.

## 5. Bot-Shaped Traffic Passes Your Verification Checks

High click-through rates combined with low time-on-site and near-zero conversion depth are the classic bot pattern. But the more direct evidence is this: the industry's dedicated verification vendors are not reliably catching it. Adalytics found that IAS labeled confirmed bot traffic as valid human visitors 77% of the time. DoubleVerify missed 21% of bot visits in the same analysis.

Verification answers whether a given impression or click was real. It does not answer whether that real impression caused a conversion. Even if every impression you paid for was human, the attribution model can still inflate conversions by crediting the same event to multiple channels or counting view-through events as causal.

## 6. View-Through Conversions Are a Large Share of Your Total

View-through attribution credits a platform for any conversion that happens within a defined window after a user was shown an ad — even if the user never clicked, never engaged, and may not have noticed the ad. A 24-hour view-through window on a high-reach network will capture a substantial fraction of your organic conversions simply because most users who buy were probably shown an impression at some point.

This is self-reported by the platform serving the impression. There is no independent verification that the view happened, that it was seen by a human, or that it influenced the conversion. Forbes was found to be running a secret made-for-advertising subdomain — a low-quality site packaged to look premium to buyers. Roughly $770M per quarter in ad spend flows to MFA sites. View-through attribution on MFA inventory is the clearest case: the impression happened on a page no one reads, the conversion happened elsewhere, and the platform claims credit for both.

## 7. Your Attribution Numbers Change When You Shift the Lookback Window

Open any platform's attribution settings and change the lookback window — say, from 30-day click to 7-day click. Watch your historical conversion numbers change retroactively. This is not a recalculation of ambiguous cases; it is the same events being counted or excluded based on a parameter you control.

This behavior reveals that attributed conversions are not a fixed count of verified events. They are a query result that depends on the model. If your reported performance can be adjusted upward by widening a window, the number is not anchored to reality. The fix is to count events that are verified at the point of conversion — not attributed after the fact by the platform reporting its own performance.

---

The common thread across all seven signs is that platform-reported attribution is self-reported by the parties being measured. Croupier's approach is advertiser-side verification: the advertiser signs coupons with a private key (Ed25519) and checks its own signature at conversion. A platform can report whatever it wants; the advertiser's own verification count doesn't move. The gap between the two numbers is the measurement.

---

*Croupier is a blind relay for cryptographic coupon books. [Learn more](https://github.com/kimjune01/croupier) or [request early access](https://croupier.ad).*
