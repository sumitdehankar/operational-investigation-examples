# Cache Miss Pressure Amplification

## Scenario

A growing SaaS platform began experiencing intermittent API latency spikes during peak business hours.  

The engineering team initially suspected:
- database performance degradation
- insufficient infrastructure scaling
- background job interference

However, infrastructure metrics did not show consistent CPU or memory saturation across services.

The primary concern was:
- increasing API response variability
- elevated database pressure during traffic spikes
- reduced operational confidence during deployments

The system architecture included:
- .NET backend APIs
- Redis caching layer
- PostgreSQL database
- background processing workers
- Kubernetes-based deployments

---

# Initial Signals

Observed symptoms included:

- Increased p95 API latency during traffic bursts
- Sudden spikes in database read pressure
- Higher connection pool utilization
- Temporary cache hit-rate degradation
- Increased operational debugging time during incidents

Additional operational concerns:

- Latency spikes were inconsistent
- Database metrics alone did not clearly identify root cause
- Teams lacked clear visibility into cache effectiveness by endpoint
- Deployment timing occasionally appeared correlated with instability

At this stage, the system did not appear critically overloaded, but operational instability was increasing.

---

# Investigation Findings

## 1. Cache Miss Amplification

Investigation showed several high-traffic API endpoints relied heavily on cache reads for acceptable response times.

During cache invalidation windows:
- multiple concurrent requests attempted identical database fetches
- repeated cache misses amplified database pressure
- request fan-out increased rapidly during traffic bursts

This created temporary cascading pressure:
- higher database load
- slower query execution
- additional request queueing
- increased latency variability

The issue was not a complete cache failure.

The primary problem was:
poor cache miss coordination under burst traffic conditions.

---

## 2. Observability Gaps

The system had:
- infrastructure monitoring
- database dashboards
- application logs

However, investigation visibility was incomplete.

Missing operational visibility included:
- cache hit/miss metrics by endpoint
- request amplification tracing
- invalidation timing visibility
- cache warm-up behavior tracking

This significantly increased debugging effort during incidents because teams could observe downstream database pressure but not the operational trigger causing it.

---

## 3. Aggressive Cache Expiration Strategy

Several frequently accessed cache entries used synchronized expiration windows.

As a result:
- multiple hot cache keys expired simultaneously
- traffic bursts immediately after expiration created pressure amplification events

The expiration strategy unintentionally introduced synchronized instability patterns.

---

## 4. Deployment Correlation Was Indirect

Initial assumptions linked deployments directly to latency spikes.

Investigation showed deployments were not the primary root cause.

However:
- deployments occasionally triggered cache invalidation events
- temporary cold-cache behavior increased database pressure sensitivity

Deployments amplified existing operational weaknesses rather than creating entirely new failures.

---

# Operational Risks

Primary operational risks identified:

- Growing database dependency during traffic bursts
- Reduced incident debugging clarity
- Increased deployment confidence erosion
- Higher probability of cascading latency events under future scale growth

An important observation:
the system still had sufficient raw infrastructure capacity.

The larger concern was:
operational instability patterns becoming harder to reason about over time.

---

# Prioritized Recommendations

## High Priority

### Add Endpoint-Level Cache Visibility

Introduce:
- cache hit/miss metrics
- invalidation tracking
- endpoint-level cache effectiveness monitoring

Primary goal:
reduce investigation ambiguity during incidents.

---

### Reduce Synchronized Cache Expiration

Introduce:
- expiration staggering
- selective TTL randomization
- gradual refresh patterns for high-traffic cache entries

This reduces coordinated cache pressure spikes.

---

### Add Request Coalescing for Hot Cache Misses

Prevent multiple concurrent requests from repeatedly triggering identical database queries during temporary cache gaps.

This helps reduce:
- database amplification
- temporary traffic bursts
- request queue pressure

---

## Medium Priority

### Improve Cache Warm-Up Strategy

For operationally important endpoints:
- preload selected cache entries after deployments
- reduce cold-cache sensitivity during rollout windows

---

### Add Investigation-Oriented Dashboards

Create dashboards focused on:
- cache behavior
- request amplification
- operational instability indicators

rather than infrastructure metrics alone.

---

# Why Certain Changes Were Deferred

Several larger architectural changes were intentionally deferred.

These included:
- database sharding
- major cache layer redesign
- service decomposition initiatives

Reasoning:
the investigation did not show evidence that architectural scaling limits had yet been reached.

Most instability originated from:
- operational coordination gaps
- observability limitations
- cache behavior under burst traffic

Lower-risk operational improvements were expected to provide significantly better short-term reliability gains.

---

# Expected Outcome

Expected improvements after prioritized changes:

- Reduced latency variability during traffic bursts
- Lower database pressure amplification
- Improved deployment confidence
- Faster operational investigations
- Better visibility into cache-related instability patterns

Most importantly:
improved operational clarity for engineering teams responding to future incidents.
