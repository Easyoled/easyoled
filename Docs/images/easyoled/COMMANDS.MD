
**Group suffix semantics:**
- `N` (immediate) → send commands immediately on SPACE
- `Nda` (delayed) → send after 3 s (or custom delay via special id "45")
- `Nd` (loop) → send in a loop every 3 s

**Numeric root rule:** `5`, `5da`, `5d` all belong to group `5`. Same-step siblings share the root.

### 7.2 Multi-Page Demo Pattern

For a full video walkthrough:

1. **Cycle 1 (pages 0-9)** — no `CLEAR`. Each step: `PAGE,N` + objects. All 10 pages stay populated for manual swipe tour.
2. **Cycle 2+ (pages 0-9)** — each step: `PAGE,N` + `CLEAR` + objects. Each page cleaned before next object.

Python should **NOT** auto-send `CLEAR` — it must come from CSV.

### 7.3 Recommended Test Order

1. `PAGE,N` first (choose page)
2. `CLEAR` (if needed)
3. Object create commands
4. `SETSTATE`/`UPDATE`/`LINK` as applicable
5. `d`-suffix for cyclic demos (e.g. `UPDATE` loop for Gauge, Wave data)

### 7.4 Coordinate Assumptions

For 1080 × 1920 portrait:
- 2-column grid: `x=100` and `x=580`
- Row spacing: ≥100 px between object groups
- Text size ≤ 70 px to fit most boxes
- Bottom safe area (last 300 px) often used by system UI — place important elements above y=1600

### 7.5 Object Interdependencies

- `LINK,textId,targetId,prop` requires both objects on the same active page at LINK time
- `VU_SOURCE` requires SEGMENTBAR objects to exist first
- `MP_TOGGLE` from ActionButton requires a MEDIAPLAYER object with matching id
- `RADIO_LOAD` requires valid `.m3u` file
- `GROUP_ADD` requires GROUP to exist first

### 7.6 Event Capture

Simulator should display app → Arduino events in a scrolling log (last 30 lines visible is a good default). Events can arrive interleaved with `ACK_OK` messages.

---

*End of document. Version 3.6.0 · Last updated: 2026*
