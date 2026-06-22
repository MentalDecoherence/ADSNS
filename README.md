# ADSNS — Autonomous Deep Space Navigation Standard

> A fully Earth-independent navigation system for interplanetary and interstellar spacecraft - think GNSS, but the "satellites" are millisecond pulsars, radio pulsars, and quasars.
> 

[](https://img.shields.io/badge/status-in%20development-blue) [](https://img.shields.io/badge/python-3.x-blue) [](https://img.shields.io/badge/license-MIT-green)

---

## Why this exists

GPS works because Earth is surrounded by satellites broadcasting precise time signals. The moment a spacecraft leaves that bubble - heading to Mars, the outer planets, or beyond - that infrastructure disappears. Current deep-space missions fall back on the **Deep Space Network**, a chain of ground antennas that triangulates a spacecraft's position from Earth. That works, but it's slow (light-time delay alone can be hours), bandwidth-limited, and entirely dependent on Earth being available and listening.

**ADSNS removes Earth from the loop.** The universe already broadcasts its own precision timing signals - we just have to listen to them the right way.

## Core idea

| Source | Function |
| --- | --- |
| **MSPs** (millisecond pulsars) | High-precision timing references — rotate with clock-like stability, used the way GNSS uses atomic clocks |
| **Pulsars** | Coarse spatial triangulation anchors |
| **Quasars** | Fixed inertial frame calibration points — distant enough to act as a stable celestial reference frame |

By measuring the **time-of-arrival (TOA)** of pulses from several of these sources and comparing them against a predicted model, a spacecraft can solve for its own position and velocity with no ground station required.

## Mathematical model

At its core, ADSNS treats navigation as a constrained spacetime optimization problem:

$\min_{x(t)} \sum_i \left| TOA^i_{measured} - TOA^i_{model}(x,t) \right|^2$

where:

- $x(t)$ — the spacecraft's state vector (position, velocity, and clock bias as a function of time)
- $TOA^i_{measured}$ — the observed time-of-arrival of pulses from source $i$
- $TOA^i_{model}(x,t)$ — the predicted time-of-arrival, computed from the source's known ephemeris and the spacecraft's hypothesized state

The model term accounts for light-travel time, the spacecraft's relative motion, and relativistic effects (time dilation, gravitational redshift) - which is why relativistic corrections live in their own dedicated module rather than being folded into the geometry code. Solving this minimization over time, rather than at a single instant, is what turns a one-shot fix into a continuous navigation filter.

## How it works (pipeline)

1. **Detection** — raw photon/pulse events are captured (`detector/photon.py`).
2. **Identification** — each detected signal is matched to a known catalog source (`detector/identifier.py`) using reference data in `som/data/pulsars.json` and `som/data/ext_sources.json`.
3. **State estimation** — TOAs are converted into a spacecraft state estimate (`navigation/signal_to_state.py`), refined through a filter (`navigation/filter.py`), with relativistic and coordinate corrections applied via `core/relativity.py` and `core/coordinates.py`.
4. **Maneuvering** — estimated state feeds into correction logic (`navigation/maneuvering.py`).
5. **Packetization** — results are serialized for downstream use via `packet/packet.py`, following the schema in `packet/packet_structure.json`.


## Data sources

ADSNS draws on publicly available pulsar and astrometric catalogs (e.g. ATNF Pulsar Catalogue-style data) stored under `som/data/`. `catalog.py` and `schema.py` handle loading and validating these against expected formats so bad reference data fails loudly instead of corrupting a state estimate silently.

## Acknowledgments

Built for the Stardance Challenge by Hack Club, in partnership with NASA. Pulsar/ephemeris reference concepts informed by public NASA mission data and the broader pulsar-navigation research literature.

## License

MIT - [see LICENSE](https://mit-license.org).
