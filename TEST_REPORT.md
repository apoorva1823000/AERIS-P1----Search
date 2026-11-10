# AERIS Part 1 — Integration Test Report

## Scope

This report validates the final Part 1 deterministic search implementation used by the Streamlit application.

## Test summary

- Focused integration suite: **150 / 150 passed**
- Randomized regression suite: **3,423 / 3,423 passed**
- Python syntax compilation: **passed**

## Tested behaviour

### Scenario generation
- Earthquake, Flood and Cyclone deterministic worlds
- Multiple seeds and parameter combinations
- All configured delivery destinations

### Search integration
- BFS never traverses a known blocked road
- UCS never traverses a known blocked road
- A* never traverses a known blocked road
- Unreachable destinations return no route rather than a blocked route
- H1 A* agrees with UCS on optimal path cost
- H1-H4 A* agree with UCS on optimal path cost whenever a route exists

### Heuristic validation
- H1 Zero: admissible and consistent
- H2 Geometric lower bound: admissible and consistent
- H3 Relaxed-network: admissible and consistent
- H4 = max(H2,H3): verified node-by-node
- H4 remains admissible and consistent in tested graphs

### Cross-module state behaviour
- Blocking an edge used by the current route forces the A* route to change when an alternative exists
- Search and comparison routines now use the same blocked-road semantics
- Heuristic calculations use the same modified graph as route planning

### External map support
- Bundled JSON maps validate
- Bundled CSV node/edge/request files validate
- Missing Depot is rejected
- Empty request sets are rejected
- Imported blocked roads are excluded from routing

## Issues found and corrected

1. **BFS blocked-road inconsistency**
   - Previous behaviour: BFS could traverse an edge marked blocked because it treated the edge as present with a huge cost.
   - Corrected behaviour: all Part 1 search algorithms exclude blocked edges completely.

2. **Empty external request validation**
   - Previous behaviour: a map with no request/demand destination could pass validation and later break destination widgets.
   - Corrected behaviour: such maps are rejected with a clear validation error.

3. **H3 mismatch between app and notebook**
   - Previous behaviour: app H3 used travel-time-only relaxed cost while the comparison notebook used a different relaxed lower-bound edge cost.
   - Corrected behaviour: both now use the same lower-bound relaxed network definition.

4. **Duplicate Streamlit page configuration**
   - Previous behaviour: `st.set_page_config()` appeared twice, including the obsolete ambulance favicon configuration.
   - Corrected behaviour: one page configuration remains, using the custom AERIS favicon.

## H3 definition after correction

For every traversable edge:

```text
c_relaxed(e) = min(c_original(e), geometric_length(e))
```

The H3 value is the shortest-path cost to the goal in this relaxed graph. Since every relaxed edge cost is no greater than the corresponding original edge cost, H3 is a lower bound on the original remaining path cost.

## Status

**Part 1 search logic passes the current regression suite and is ready for redeployment.**

Live browser rendering was not automated from this environment because the Streamlit public host and GitHub host block the available fetch layer. Logic testing was performed against the exact publish-ready source package used to create the repository.
