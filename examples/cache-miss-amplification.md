# Cache Miss Amplification Under Peak Usage

## Scenario

A backend platform began experiencing intermittent latency spikes during peak usage periods.

Observed symptoms included:

* elevated API response times
* increased database CPU utilization
* dashboard load inconsistencies
* growing request queue times during traffic spikes

Initial assumptions inside the engineering team focused primarily on database scaling concerns.

---

## Initial Operational Signals

Initial observations showed:

* request queue times increasing before database saturation became critical
* cache miss rates spiking during peak traffic windows
* several dashboard aggregation endpoints bypassing cache for freshness requirements
* cache TTL configured aggressively low for some expensive aggregation paths

At first glance, database pressure appeared to be the primary bottleneck.

However, further investigation suggested that increased database load was partially being amplified by reduced cache effectiveness.

---

## Investigation Focus Areas

The investigation focused on:

* cache hit/miss behavior
* workload amplification patterns
* dashboard aggregation workflows
* freshness requirements
* database query pressure
* background workload timing

---

## Key Observations

### Cache Strategy Misalignment

Not all workloads required the same freshness guarantees.

Some expensive aggregation queries were configured with:

* short cache TTL values
* aggressive invalidation behavior
* cache bypassing for near-real-time responses

This caused avoidable increases in repeated database queries during traffic spikes.

---

### Aggregation Cost Amplification

Several dashboard endpoints relied on expensive aggregation queries across large transaction datasets.

Although portions of the aggregated data changed frequently, other portions remained relatively stable.

The caching strategy treated the entire aggregation workload as highly volatile, increasing unnecessary database pressure.

---

### Queue Time Behavior

Queue wait times increased significantly during traffic spikes.

This suggested that request contention and resource saturation were amplifying latency beyond pure query execution time.

The database was absorbing downstream pressure created by workload amplification patterns.

---

## Risk Evaluation

Several potential remediation paths were considered.

### Immediate Infrastructure Scaling

Temporary vertical database scaling was considered operationally safe as a short-term mitigation strategy if pressure continued increasing.

However, infrastructure scaling alone would not address workload amplification inefficiencies.

---

### Large Architectural Changes

Separating workloads or restructuring infrastructure was intentionally deprioritized initially because:

* operational visibility was still incomplete
* deployment risk was higher
* rollback complexity would increase
* root workload behavior was not fully understood yet

---

## Prioritized Recommendations

### 1. Improve Cache Segmentation Strategy

Separate caching behavior based on actual freshness requirements rather than applying uniform TTL behavior.

Examples:

* near-real-time workloads with short TTL
* semi-static aggregation workloads with longer TTL
* selective cache invalidation instead of full bypassing

---

### 2. Review Expensive Aggregation Queries

Investigate:

* indexing opportunities
* unnecessary query complexity
* aggregation workload distribution
* repeated computation patterns

---

### 3. Shift Background Workloads Away From Peak Usage

Some scheduled workloads overlapped with high traffic periods.

Adjusting workload timing represented a low-risk operational improvement with minimal architectural impact.

---

### 4. Improve Observability Visibility

Additional metrics around:

* cache effectiveness
* queue wait times
* request contention
* aggregation latency

would improve future investigation clarity.

---

## Final Observation

The investigation highlighted that database saturation was not acting as an isolated bottleneck.

Operational pressure was being amplified by workload behavior, cache strategy alignment issues, and traffic-time contention patterns.

Low-risk operational improvements were prioritized before larger architectural changes in order to reduce instability risk while improving investigation clarity.
