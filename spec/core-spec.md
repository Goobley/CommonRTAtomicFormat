CRTAF v0.1.0
============

Common Radiative Transfer Atomic Format.

Objectives
----------

CRTAF aims to provide a flexible interchange format for atomic data used in radiative transfer (RT) models such as [Lightweaver](https://github.com/Goobley/Lightweaver), [RH](https://github.com/ITA-Solar/rh), and MULTI.
Many of these frameworks have structured ad hoc input formats that require custom parsers, creating barriers to interchange of models between these tools.

CRTAF aims to set out a set of core features that can be easily implemented by existing RT codes, but remains extensible by different authors.
This is achieved by having a minimalistic set of core features, along with a flexible set of extensions.
The official Python [`crtaf`](#TODO) package will implement all core extensions as well as user contributed extensions.
The CRTAF format then represents a two-level format, a high-level representation of the data, and a simplified format utilising the minimal set of extensions necessary to specify the atomic model.
The official `crtaf` package will serve as the reference implementation to convert between high-level and simplified CRTAF formats, though other package may appear with adoption of the format.

This specification is inspired and based on work by Tiago Pereira in [Muspel](https://github.com/tiagopereira/Muspel.jl) and [AtomicData](https://github.com/tiagopereira/AtomicData.jl/), in addition to the existing model atom formats of RH, MULTI, Snapi, and the work on "atoms as code" in Lightweaver.

Notes
-----

* When discussing atomic transitions the integer indicies i and j are used with j > i.

Spec
----

* CRTAF models are stored in a valid [TOML](https://toml.io) file.
* CRTAF models will contain a `CRTAF` table specified in [CRTAF Table](#crtaf-table).
* CRTAF models will contain an `extensions` array specified in [Extensions array](#extensions-array).
* CRTAF models will contain units where possible. The simplified format will use a pre-defined set of units (SI with wavelengths in nm) for every field to simplify parsing.

CRTAF Table
-----------

The CRTAF table will be a top-level table with key `CRTAF`.

* It must specify the version of the CRTAF spec the model complies to.
* It must specify the `level` as either `"high-level"` or `"simplified"`
* It must contain an extension array (potentially empty), as specified in [Extensions Array](#extensions-array).
* It may contain a multi-line string of `notes` from the model author.

Example:

```toml
[CRTAF]
version = "v0.1.0"
level = "high-level"
notes = '''
This models was designed for the calculation of the Mg ii IRIS lines using Lightweaver.
'''
extensions = []
```

### Extensions Array

The `CRTAF` table must contain an array with key `extensions` this array contains the name of zero or more `CRTAF` extensions that are necessary to parse the provided file.
All extension names should be provided as strings.
The available extensions are detailed in the extensions documentation.

Core Atomic Data
----------------

### High-level format

The file will provide a top-level `element` table.
This table will contain the element `symbol` (string), and the `atomic_mass` (in amu).
The element table may also specify atomic numbers `N` and `Z` as integers, allowing for multiple isotopes of species.
If `N` is provided the model should be considered as a model of a particular isotope, and not a general case for a species.

The `abundance` of the species may also be specified (in the conventional logarithmic scale used by Asplund et al where $\log \epsilon_H = 12$ and $\log \epsilon_X = \log(N_X / N_H) + 12$ where $N_X$ and $N_H$ are the number densities of a given element X and H respectively).
Abundance is provided in an advisory capacity: if a code has a different way of specifying elemental abundance it may overrule the behaviour in the atom file.
This behaviour must be documented in the particular CRTAF implementation.

Example:
```toml
[element]
symbol = "Ca"
atomic_mass = 40.078
abundance = 6.34
```

### Simplified format

In the simplified CRTAF format _all_ of the above fields with the exception of `N` are required. If `N` is provided and the code has no facility for processing isotopes, an warning should be emitted.

Atomic Levels
-------------

### High-level format

The file will provide a top-level `atomic_levels` table.
This table will contain a dynamic number of subtables corresponding to the atomic energy levels present in the model.
**These will be processed by the parser in lexicographic order of the keys, not the order of definition.**

The table for each atomic energy level must contain:

* An `energy` table containing a `value` for the energy (float), and the `unit` used (string)
* The statistical weight value `g` (int)
* The ionisation `stage` (int). This is chosen so the neutral element has stage 1 to correspond to typical astronomy notation (e.g. Ca I for neutral Ca).

The table for each atomic energy level may also contain:

* A `label` (string)
* `J` total angular momentum quantum number as either a table containing `numerator` and `denominator` as int (representing a rational fraction) or a single float. ($g = 2J+1 = (2S+1)(2L+1)$).
* `L` orbital angular momentum quantum number (int)
* `S` spin quantum number (fraction or float, as `J`)

For more information on the meaning of the quantum numbers and typical level labels see [Kaastra et al](https://ned.ipac.caltech.edu/level5/Sept08/Kaastra/Kaastra2.html) or the overview on [Wikipedia](https://en.wikipedia.org/wiki/Term_symbol).

If any of `J`, `L`, or `S` are present, they must all be present (for a given level).

Example:

```toml
[atomic_levels.ground]
energy = { value = 0.0, unit = "cm^-1" }
g = 2
stage = 2
label = "Ca II 3P6 4S 2SE"
J = {numerator = 1, denominator = 2}
L = 0
S = {numerator = 1, denominator = 2}

[atomic_levels.first]
energy = { value = 13650.19, unit = "cm^-1" }
g = 4
stage = 2
label = "Ca II 3P6 3D 2DE 3"
J = {numerator = 3, denominator = 2}
L = 2
S = {numerator = 1, denominator = 2}
```

### Simplified format

The simplified format of each atomic level table is the same as the high-level format with the additional requirements:

* `energy` is in typical spectroscopic units `"cm^-1"`
* `energy_eV` table is present with units `"eV"` and the `value` of the level energy in eV.

Spectral Lines
--------------

The file will provide a top-level `radiative_bound_bound` array. This array will contain the specification of each spectral line in the model as a table.

### High-level format

The table for each bound-bound transition must contain:

* The transition `type` (string). The core spec provides support for `"Voigt"` and `"PRD-Voigt"`. Extensions may override the fields present and parsing of this table.
* The key `transition` with value an array of two atomic level labels specifying the upper and lower levels of the transition by label.
* The dimensionless oscillator strength `f_value` (float).
* An array of `broadening` tables. These contain a `type` (string) and are specified in [Line Broadening](#line-broadening).
* A subtable describing the `wavelength_grid` containing the wavelength grid `type` (string). The core spec provides support for `"Linear"` and `"Tabulated"`, specified in [Line Wavelength Grids](#line-wavelength-grids). Extensions may override the fields present and parsing of this table.

### Simplified format

The simplified format for a bound-bound transition is the same as the high-level format, with the the substitution of the simplified format of `broadening` and `wavelength_grid` and the addition of:

* `Aji`: table of `unit` (string - `"s^-1"`) and `value` (float). The Einstein A spontaneous emission coefficient.
* `Bji`: table of `unit` (string - `"m^2 * Hz / J"`) and `value` (float). The Einstein B stimulated emission coefficient per frequency following the Mihalas solid angle definition.
* `Bji_wavelength`: table of `unit` (string - `"m^3 / J"`) and `value` (float). The Einstein B stimulated emission coefficient per wavelength following the Mihalas solid angle definition.
* `Bij`: table of `unit` (string - same as `Bji`) and `value` (float). The Einstein B induced absorption coefficient per frequency.
* `Bij_wavelength`: table of `unit` (string - same as `Bji`) and `value` (float). The Einstein B induced absorption coefficient per wavelength.
* `lambda0`: table of `unit` (string - `"nm"`) and `value` (float). The rest wavelength of the transition.

[Hilborn (2002)](https://arxiv.org/abs/physics/0202029) provides a good overview of the different possible expressions for Einstein B coefficients.

Additionally, the order of atomic level labels in `transition` is sorted to be `[upper, lower]`.

Line Broadening
---------------

### High-level format

Each broadening table must contain a key `type` with associated string value.
The core spec requires support for types:

* `"Natural"`: must provide `value` (float) and (`unit` (string) - typically `"s^-1"`).
* `"Stark_Linear_Sutton`: only valid for hydrogen. May provide `n_upper` (int), and `n_lower` (int). Otherwise these will be inferred from the atomic model, under the assumption that each level represents a successive principal quantum number `n`.
* `"Stark_Quadratic"`: May provide `scaling` (float), linear scaling coefficient - assumed to be 1.0 if not present, and `C_4` as a subtable of `value` (float) and `unit` (string) - typically `"m^3 * s^-1"`. If `C_4` is not provided, it must be computed following a common approximation (e.g. Traving 1960).
* `"VdW_Unsold"`: May provide `H_scaling` (float) and `He_scaling` (float) which scale the terms related to hydrogen and helium respectively. These are assumed to be 0.0 if not present.
* `"Scaled_Exponents"`: Computes broadening of the form "scaling * T^a * n_H^b * n_e^c" (where T is temperature, n_H is neutral H density and n_e is electron density). Must provide:
    * `scaling` (float)
    * `temperature_exponent` (float)
    * `hydrogen_exponent` (float)
    * `electron_exponent` (float)

### Simplified format

The simplified format requires support for `"Natural"` and `"Scaled_Exponents"`. For `"Natural"` the units must be `"s^-1"`.

Line Wavelength Grids
---------------------

### High-level format

The `wavelength_grid` table of a spectral line describes the wavelength quadrature provided as input into the program.
The code is not required to use this wavelength grid, but it must indicate if it does not.

The core spec requires support for the `"Linear"` and `"Tabulated"` wavelength grid types.
These must provide:

* `"Linear"`:
    * `n_lambda` (int): the number of wavelength points (odd).
    * `delta_lambda`: table of `value` (float) and `unit` (string) specifying the half-width of the transition from the line rest wavelength.
* `"Tabulated"`:
    * `unit` (string): The unit of `lambda`
    * `lambda`: Array of wavelength points to sample at (relative to the line rest wavelength).

### Simplified format

The simplified format is the same as the high-level format, with the additional requirements:

* `"Linear"`:
    * `unit`: Must be in `"nm"`
* `"Tabulated"`:
    * `unit`: Must be in `"nm"`

Continua
--------

The file will provide a top-level `radiative_bound_free` array. This array will contain the specification of each atomic continuum as a table.

### High-level format

The table describing each atomic continuum must contain:

* The continuum `type` (string). The core spec provides support for `"Hydrogenic"` and `"Tabulated"`. Extensions may override the fields present and parsing of this table.
* The key `transition` with value an array of two atomic level labels specifying the upper and lower levels of the transition by label.

Depending on the value of `type` additional keys will be present:
* `"Hydrogenic"`: A typical hydrogenic continuum scaling as frequency cubed.
    * `unit`: array of two strings specifying the wavelength and cross-section units respectively (typically `["nm", "m^2"]`).
    * `sigma_peak`: A table of `unit` (string - typically "m^2") and `value` (float). The maximal cross section of the continuum (at the edge).
    * `lambda_min`: A table of `unit` (string - typically "nm") and `value` (float). The minimum wavelength to compute the continuum for.
    * `n_lambda` (int): The number of wavelength samples between the edge wavelength and `lambda_min`.

* `"Tabulated"`: A tabulated continuum.
    * `unit`: array of two strings specifying the wavelength and cross-section units respectively (typically `["nm", "m^2"]`).
    * `value`: An array of arrays containing two float entries each: the wavelength of the sample, and the cross-section at that wavelength.

### Simplified format

* All in `"nm"` and `"m^2"`.
* Interpolation of cross-sections is implementation defined.

Collisional Rates
-----------------


Extensions
----------

Ratified extensions specifications are present in the [extensions directory](./extensions).
Extension specifications are organised by category:

| Prefix | Category                         |
|:------:|----------------------------------|
| 00     | Meta (CRTAF table)               |
| 10     | Atomic Levels                    |
| 20     | Spectral Lines (type)            |
| 21     | Spectral Lines (broadening)      |
| 22     | Spectral Lines (wavelength grid) |
| 30     | Continua                         |
| 40     | Collisional Rates                |

The extension filename must be of the form `{numerical_prefix}-{extension_name}.md`, where the `extension_name` has words split by underscores.
For example [22-linear_core_exp_wings.md](./extensions/22-linear_core_exp_wings.md
).
This `extension_name` is to be added to the [extensions array](#extensions-array) of any model it is used in.
