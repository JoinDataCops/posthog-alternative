# PostHog Alternatives & Analytics Stack 2026: A Practical README

Comparison reference for engineers evaluating PostHog and the alternatives in 2026.

## TL;DR

| Tool | Best for | Setup | Free tier | Bot filter | CAPI |
|---|---|---|---|---|---|
| PostHog | Full product analytics suite | 1 hr | 1M events / 5K replays | No | No |
| Mixpanel | Focused product analytics | 1 hr | 100K MMU | No | No |
| Amplitude | Enterprise product analytics | 4 hr | Limited | No | No |
| Heap | Auto-capture | 30 min | Limited | No | No |
| Statsig | Feature flags + experiments | 1 hr | 1M events | No | No |
| Plausible | Privacy-first pageviews | 30 min | None (paid only) | Light | No |
| Fathom | Privacy-first pageviews | 30 min | None (paid only) | Light | No |
| DataCops | First-party tracking + CAPI + bot filter | 18 min | 2K sessions | Yes | Yes |

## Why this README exists

The conversation around analytics in 2026 isn't the same as 2022. Bots are 51% of all web traffic per Imperva 2025 (37% bad bots specifically). Ad blockers plus ITP block 10 to 30% of client-side events on default install per PostHog's own docs. Meta's average IVT is 8.20%.

Pure product analytics tools don't address any of those. They capture what fires. This README covers the alternatives plus the trust-infrastructure layer underneath.

## Key concepts

**Client-side vs server-side capture**: Client-side runs JavaScript that sends events to a tracking domain. Ad blockers see the domain and block it. Server-side captures events at the application layer and forwards to vendors. CNAME-based first-party (DataCops) is hybrid: client-side capture but on your own subdomain, so blockers don't see a third-party.

**Reverse proxy**: workaround for client-side blocking. Route the analytics requests through your own domain so they look first-party. PostHog supports it, and notes 10 to 30% event recovery when enabled. Works at the cost of meaningful DevOps overhead.

**CAPI (Conversion API)**: server-side delivery of conversion events to ad platforms (Meta, Google, TikTok, LinkedIn). Recovers events lost to client-side blocking. Different surface than product analytics.

**Bot filtering**: pre-ingestion. Drops bot traffic before it pollutes dashboards or CAPI feeds. Most product analytics tools don't ship this.

## Architecture choice

**Pattern A: Full product analytics suite (PostHog, Mixpanel, Amplitude)**

One tool covers funnels, retention, cohorts, replay, flags.

Pros: Unified dashboard. One vendor.
Cons: Pricing scales with breadth even if you don't use it all. No CAPI. No bot filter. Subject to client-side blocking.

**Pattern B: Lean privacy-first (Plausible, Fathom)**

Pageview-focused. Privacy posture clean. No banner needed in many jurisdictions.

Pros: Cheap. Fast. Compliant by design.
Cons: No funnels (Plausible paywalls them). No replay. No CAPI.

**Pattern C: Trust-infrastructure layer (DataCops) underneath whatever analytics**

CNAME on your subdomain captures everything first-party, filters bots, forwards clean events to your analytics tool plus CAPI plus consent state.

Pros: Fixes the input-quality problem at the source. Works alongside any analytics tool.
Cons: Not a product analytics suite. SOC 2 Type II in progress.

## Install paths

### PostHog (cloud)

```javascript
import posthog from 'posthog-js'
posthog.init('YOUR_KEY', { api_host: 'https://us.posthog.com' })
posthog.capture('event_name', { property: 'value' })
```

Time to live: ~1 hour.

### PostHog with reverse proxy (recommended for prod)

Set up Cloudflare Worker or nginx config to proxy `/_ph` to PostHog. Configure SDK to use your domain. Recovers 10 to 30% of events. ~1 day of work.

### DataCops

```bash
# 1. CNAME: datacops.yourdomain.com -> cdn.yourdomain.com
# 2. <script src="https://datacops.yourdomain.com/dc.js"></script>
# 3. Configure forwarders in dashboard (PostHog, Mixpanel, Meta CAPI, etc.)
```

Time to live: 18 minutes.

## Observability checklist

Whatever stack you pick, your analytics layer should answer these in 5 minutes:

1. What's the event capture rate (% of expected events actually captured)?
2. What's the bot rate in captured events?
3. What's the ad-blocker loss estimate?
4. For ad campaigns, what's the CAPI delivery rate to Meta/Google?
5. What's the consent state per session?

If you can't answer these, you're flying blind on input quality.

## Cost model at 5M events/mo

- PostHog: $300+ (after free tier)
- Mixpanel: ~$1,000 (Growth tier)
- Amplitude: ~$2,000 (custom)
- Heap: custom
- Statsig: ~$300 (Growth)
- Plausible: $39 (pageview-equivalent volume)
- DataCops Business: $49 (covers 5M events for the input layer; analytics tool sits alongside)

The bundling argument: keep PostHog at the analytics layer, add DataCops at the input layer for ~$49 to recover the 30% events lost to blockers and filter the 7.4% bot baseline. Cheaper than upgrading PostHog tier or self-hosting.

## Failure modes

- **Ad blocker spike**: 30% events lost overnight when uBlock pushes a list update. Mitigation: first-party CNAME.
- **ITP storage clear**: Safari rotates the storage on a 7-day timer. Mitigation: server-side ID resolution.
- **Vendor outage**: PostHog cloud down. Mitigation: graceful event buffering on client + forwarder retry.
- **Bot spike**: scraper campaign floods events. Mitigation: pre-ingestion bot filter.
- **PostHog cost spike**: hit the next tier mid-month. Mitigation: rate limiting + alerting on event volume.

## Decision pseudo-code

```python
def pick_analytics_stack(profile):
 if profile == "full_product_analytics_breadth":
 return "PostHog"
 if profile == "focused_product_analytics":
 return "Mixpanel"
 if profile == "enterprise_product_analytics":
 return "Amplitude"
 if profile == "feature_flags_experiments_focus":
 return "Statsig"
 if profile == "privacy_first_pageviews":
 return "Plausible or Fathom"
 if profile == "fix_input_quality_keep_analytics_tool":
 return "DataCops underneath whatever you have"
 if profile == "first_party_tracking_plus_capi_plus_consent_in_one":
 return "DataCops alone"
 return "evaluate based on the breadth question and input-quality goals"
```

## Further reading

- Full long-form comparison: https://joindatacops.com/blog/datacops-vs-posthog
- DataCops product context: https://joindatacops.com/first-party-analytics
- PostHog pricing: https://posthog.com/pricing
- PostHog ad-blocker docs: https://posthog.com/docs/advanced/proxy

## Contributions

Open an issue with source URL and date if a vendor changes pricing or capabilities.

License: MIT for the doc.

---

Research by [DataCops](https://www.joindatacops.com) · First-party tracking, consent infrastructure & fraud prevention.
