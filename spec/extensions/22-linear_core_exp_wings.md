Linear Core Exp Wings
=====================

| Extension Name            | Extension Type |
|---------------------------|-----------------|
| `"linear_core_exp_wings"` | Wavelength Grid |

Purpose
-------

This extension adds the ubiquitous wavelength grid used in RH and Lightweaver (amongst others), where the quadrature in the line core is linearly spaced, and the wings are exponentially spaced.

Definition
----------

### High-Level Format

This wavelength grid is designated by `type` "LinearCoreExpWings", and also requires:
* `q_core`: the distance from the line core to the edge of the the linearly spaced section (float, in Doppler units)
* `q_wing`: the distance from the line core to the edge of grid (float, in Doppler units)
* `n_lambda`: the number of wavelength points (int). If even, one point is removed to place a sample on the line core and symmetrically on both sides.
* `v_micro_char`: mapping of `unit` (string) typically "km / s", and `value` (float). This describes the characteristic microturbulent velocity to consider for the conversion between Doppler units and wavelength.

### Simplified Format

* Simplifies to a `TabulatedGrid` from the core spec, using the visitor provided in the reference implementation.

Notes
-----

The reference implementation follows that of RH and Lightweaver directly (tested against Lightweaver). If `q_wing <= 2 * q_core` a linear spacing is used throughout the grid.