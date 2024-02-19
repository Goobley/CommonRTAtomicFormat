MULTI Wavelength Grid
=====================

| Extension Name            | Extension Type |
|---------------------------|-----------------|
| `"multi_wavelength_grid"` | Wavelength Grid |

Purpose
-------

This extension adds the wavelength grid used in MULTI (amongst others), where the quadrature in the line core is linearly spaced in Doppler units and is linearly spaced in log(Doppler units) in the wings.

Present in Muspel, can be traced back from MULTI to a routing by Åke Nordlund.

Definition
----------

### High-Level Format

This wavelength grid is designated by `type` "MULTI", and also requires:
* `q0`: the distance from the line core to the edge of the the linearly spaced section (float, in Doppler units)
* `qmax`: the distance from the line core to the edge of grid (float, in Doppler units)
* `n_lambda`: the number of quadrature points (int). If even, one point is added to place a sample on the line core and symmetrically on both sides.
* `q_norm`: mapping of `unit` (string) typically "km / s", and `value` (float). This describes the characteristic microturbulent velocity to consider for the conversion between Doppler units and frequency.

### Simplified Format

* Simplifies to a `TabulatedGrid` from the core spec, using the visitor provided in the reference implementation.

Notes
-----

* The reference implementation tested against Muspel.
* If `qmax <= q0` a linear spacing is used throughout the grid.
* This implementation converts from Doppler units to frequency, and so is not 100% symmetric/equidistant in wavelength.