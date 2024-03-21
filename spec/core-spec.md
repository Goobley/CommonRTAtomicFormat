CRTAF v0.2.0
============

Common Radiative Transfer Atomic Format.

Objectives
----------

CRTAF aims to provide a flexible interchange format for atomic data used in radiative transfer (RT) models such as [Lightweaver](https://github.com/Goobley/Lightweaver), [RH](https://github.com/ITA-Solar/rh), and MULTI.
Many of these frameworks have structured ad hoc input formats that require custom parsers, creating barriers to interchange of models between these tools.

CRTAF aims to set out a set of core features that can be easily implemented by existing RT codes, but remains extensible by different authors.
This is achieved by having a minimalistic set of core features, along with a flexible set of extensions.
The official Python [`crtaf`](https://github.com/Goobley/crtaf-py) package will implement all core extensions as well as user contributed extensions.
The CRTAF format then represents a two-level format, a high-level representation of the data, and a simplified format utilising the minimal set of extensions necessary to specify the atomic model.
The official `crtaf` package will serve as the reference implementation to convert between high-level and simplified CRTAF formats, though other package may appear with adoption of the format.

This specification is inspired and based on work by Tiago Pereira in [Muspel](https://github.com/tiagopereira/Muspel.jl) and [AtomicData](https://github.com/tiagopereira/AtomicData.jl/), in addition to the existing model atom formats of RH, MULTI, Snapi, and the work on "atoms as code" in Lightweaver.

Notes
-----

* When discussing atomic transitions the integer indices i and j are used with j > i.

Spec
----

* CRTAF models are stored in a valid [YAML](https://yaml.org) file. This file is used as a more user-friendly variant of JSON, and the format does not make use of advanced YAML features.
* CRTAF models will contain a `crtaf_meta` mapping specified in [CRTAF Mapping](#crtaf-table).
* CRTAF models will contain an `extensions` array specified in [Extensions array](#extensions-array).
* CRTAF models will contain units where possible. The simplified format will use a pre-defined set of units (SI with wavelengths in nm) for every field to simplify parsing.

CRTAF Mapping
-------------

The CRTAF mapping will be a top-level mapping with key `mapping`.

* It must specify the version of the CRTAF spec the model complies to as a string.
* It must specify the `level` as either `"high-level"` or `"simplified"`
* It must contain an extension array (potentially empty), as specified in [Extensions Array](#extensions-array).
* It may contain a multi-line string of `notes` from the model author.

Example:

```yaml
crtaf_meta:
  version: "v0.2.0"
  level: high-level
  extensions: []
  notes: A test.
```

### Extensions Array

The `CRTAF` table must contain an array with key `extensions` this array contains the name of zero or more `CRTAF` extensions that are necessary to parse the provided file.
All extension names should be provided as strings.
The available extensions are detailed in the extensions documentation.

Core Atomic Data
----------------

### High-level format

The file will provide a top-level `element` mapping.
This mapping will contain the element `symbol` (string), and the `atomic_mass` (float, in amu).
The element table may also specify atomic numbers `N` and `Z` as integers, allowing for multiple isotopes of species.
If `N` is provided the model should be considered as a model of a particular isotope, and not a general case for a species.

The `abundance` of the species may also be specified (in the conventional logarithmic scale used by Asplund et al where $\log \epsilon_H = 12$ and $\log \epsilon_X = \log(N_X / N_H) + 12$ where $N_X$ and $N_H$ are the number densities of a given element X and H respectively).
Abundance is provided in an advisory capacity: if a code has a different way of specifying elemental abundance it may overrule the behaviour in the atom file.
This behaviour must be documented in the particular CRTAF implementation.

Example:
```yaml
element:
  symbol: Ca
  atomic_mass: 40.005
  abundance: 6.0
  Z: 20
```

### Simplified format

In the simplified CRTAF format _all_ of the above fields with the exception of `N` are required. If `N` is provided and the code has no facility for processing isotopes, an warning should be emitted.

Atomic Levels
-------------

### High-level format

The file will provide a top-level `levels` mapping.
This mapping will contain a dynamic number of mappings corresponding to the atomic energy levels present in the model.

The table for each atomic energy level must contain:

* An `energy` mapping containing a `value` for the energy (float), and the `unit` used (string)
* The statistical weight value `g` (int)
* The ionisation `stage` (int). This is chosen so the neutral element has stage 1 to correspond to typical astronomy notation (e.g. Ca I for neutral Ca).

The mapping for each atomic energy level may also contain:

* A `label` (string)
* `J` total angular momentum quantum number as either a table containing `numerator` and `denominator` as int (representing a rational fraction) or a single float. ($g = 2J+1 = (2S+1)(2L+1)$).
* `L` orbital angular momentum quantum number (int)
* `S` spin quantum number (fraction or float, as `J`)

For more information on the meaning of the quantum numbers and typical level labels see [Kaastra et al](https://ned.ipac.caltech.edu/level5/Sept08/Kaastra/Kaastra2.html) or the overview on [Wikipedia](https://en.wikipedia.org/wiki/Term_symbol).

If any of `J`, `L`, or `S` are present, they must all be present (for a given level).

Example:

```yaml
levels:
  first:
    energy:
      unit: 1 / cm
      value: 123.0
    g: 1
    stage: 1
    label: First Level
  second:
    energy:
      unit: 1 / cm
      value: 456.0
    g: 2
    stage: 1
    label: Second Level
```

### Simplified format

The simplified format of each atomic level table is the same as the high-level format with the additional requirements:

* `energy` is in typical spectroscopic units `"1/ cm"`
* `energy_eV` table is present with units `"eV"` and the `value` of the level energy in eV.

Spectral Lines
--------------

The file will provide a top-level `lines` sequence. This array will contain the specification of each spectral line in the model as a mapping.

### High-level format

The mapping for each bound-bound transition must contain:

* The transition `type` (string). The core spec provides support for `"Voigt"` and `"PRD-Voigt"`. Extensions may override the fields present and parsing of this mapping.
* The key `transition` with value an array of two atomic level labels specifying the upper and lower levels of the transition by label.
* The dimensionless oscillator strength `f_value` (float).
* A sequence of `broadening` mappings. These contain a `type` (string) and are specified in [Line Broadening](#line-broadening).
* A sub-mapping describing the `wavelength_grid` containing the wavelength grid `type` (string). The core spec provides support for `"Linear"` and `"Tabulated"`, specified in [Line Wavelength Grids](#line-wavelength-grids). Extensions may override the fields present and parsing of this table.

Example:

```yaml
lines:
- type: PRD-Voigt
  transition: [second, first]
  f_value: 0.1
  broadening:
  - {type: Natural, elastic: false, value: {unit: 1 / s, value: 10000000.0}}
  - {type: Stark_Multiplicative, elastic: true, C_4: {unit: m3 / s, value: 7.0}, scaling: 3.0}
  - {type: Stark_Linear_Sutton, elastic: true}
  wavelength_grid:
    type: Linear
    n_lambda: 201
    delta_lambda:
      unit: nm
      value: 0.01
```

### Simplified format

The simplified format for a bound-bound transition is the same as the high-level format, with the substitution of the simplified format of `broadening` and `wavelength_grid` and the addition of:

* `Aji`: table of `unit` (string - `"1 / s"`) and `value` (float). The Einstein A spontaneous emission coefficient.
* `Bji`: table of `unit` (string - `"m2 Hz / J"`) and `value` (float). The Einstein B stimulated emission coefficient per frequency following the Mihalas solid angle definition.
* `Bji_wavelength`: table of `unit` (string - `"m3 / J"`) and `value` (float). The Einstein B stimulated emission coefficient per wavelength following the Mihalas solid angle definition.
* `Bij`: table of `unit` (string - same as `Bji`) and `value` (float). The Einstein B induced absorption coefficient per frequency.
* `Bij_wavelength`: table of `unit` (string - same as `Bji`) and `value` (float). The Einstein B induced absorption coefficient per wavelength.
* `lambda0`: table of `unit` (string - `"nm"`) and `value` (float). The rest wavelength of the transition.

[Hilborn (2002)](https://arxiv.org/abs/physics/0202029) provides a good overview of the different possible expressions for Einstein B coefficients.

Additionally, the order of atomic level labels in `transition` is sorted to be `[upper, lower]`.
This is convention in the high-level case, but guaranteed for a simplified model.

Example:
```yaml
lines:
- type: PRD-Voigt
  transition: [second, first]
  f_value: 0.1
  broadening:
  - {type: Natural, elastic: false, value: {unit: 1 / s, value: 10000000.0}}
  - {type: Scaled_Exponents, elastic: true, scaling: 21.0, temperature_exponent: 0.0,
    hydrogen_exponent: 0.0, electron_exponent: 1.0}
  - {type: Scaled_Exponents, elastic: true, scaling: 0.0, temperature_exponent: 0.0,
    hydrogen_exponent: 0.0, electron_exponent: 0.6666666666666666}
  wavelength_grid:
    type: Linear
    n_lambda: 201
    delta_lambda:
      unit: nm
      value: 0.01
  Aji:
    unit: 1 / s
    value: 14793.150991996721
  Bji:
    unit: m2 / (J s)
    value: 1008373122939940.2
  Bji_wavelength:
    unit: m3 / J
    value: 0.00303327713637598
  Bij:
    unit: m2 / (J s)
    value: 504186561469970.1
  Bij_wavelength:
    unit: m3 / J
    value: 0.00151663856818799
  lambda0:
    unit: nm
    value: 30030.030030030026
```

Line Broadening
---------------

### High-level format

Each broadening mapping must contain a key `type` with associated string value.
The core spec requires support for types:

* `"Natural"`: must provide `value` (float) and (`unit` (string) - typically `"1 / s"`).
* `"Stark_Linear_Sutton`: only valid for hydrogen. May provide `n_upper` (int), and `n_lower` (int). Otherwise these will be inferred from the atomic model, following the statistical weight of each level $g=2n^2$.
* `"Stark_Quadratic"`: May provide `scaling` (float), linear scaling coefficient - assumed to be 1.0 if not present.
* `"Stark_Multiplicative"`: May provide `scaling` (float), linear scaling coefficient - assumed to be 1.0 if not present and `C_4` as a mapping of `value` (float) and `unit` (string) - typically `"m3 / s"`. If `C_4` is not provided, it must be computed following a common approximation (e.g. Traving 1960).
* `"VdW_Unsold"`: May provide `H_scaling` (float) and `He_scaling` (float) which scale the terms related to hydrogen and helium respectively. These are assumed to be 0.0 if not present.
* `"Scaled_Exponents"`: Computes broadening of the form "scaling * T^a * n_H^b * n_e^c" (where T is temperature, n_H is neutral (ground) H density and n_e is electron density). Must provide:
    * `scaling` (float)
    * `temperature_exponent` (float)
    * `hydrogen_exponent` (float)
    * `electron_exponent` (float)

### Simplified format

The simplified format requires support for `"Natural"` and `"Scaled_Exponents"`. For `"Natural"` the units must be `"1 / s"`.

Line Wavelength Grids
---------------------

### High-level format

The `wavelength_grid` mapping of a spectral line describes the wavelength quadrature provided as input into the program.
The code is not required to use this wavelength grid, but it must indicate if it does not.

The core spec requires support for the `"Linear"` and `"Tabulated"` wavelength grid types.
These must provide:

* `"Linear"`:
    * `n_lambda` (int): the total number of wavelength points (typically odd).
    * `delta_lambda`: mapping of `value` (float) and `unit` (string) specifying the half-width of the transition from the line rest wavelength.
* `"Tabulated"`:
    * `unit` (string): The unit of `wavelengths`
    * `wavelengths`: Sequence of wavelength points to sample at (relative to the line rest wavelength as 0).

### Simplified format

The simplified format is the same as the high-level format, with the additional requirements:

* `"Linear"`:
    * `unit`: Must be in `"nm"`
* `"Tabulated"`:
    * `unit`: Must be in `"nm"`

Continua
--------

The file will provide a top-level `continua` sequence. This sequence will contain the specification of each atomic continuum as a mapping.

### High-level format

The mapping describing each atomic continuum must contain:

* The continuum `type` (string). The core spec provides support for `"Hydrogenic"` and `"Tabulated"`. Extensions may override the fields present and parsing of this mapping.
* The key `transition` with value a sequence of two atomic level labels specifying the upper and lower levels of the transition by label.

Depending on the value of `type` additional keys will be present:
* `"Hydrogenic"`: A typical hydrogenic continuum scaling as frequency cubed.
    * `sigma_peak`: A mapping of `unit` (string - typically "m^2") and `value` (float). The maximal cross section of the continuum (at the edge).
    * `lambda_min`: A mapping of `unit` (string - typically "nm") and `value` (float). The minimum wavelength to compute the continuum for.
    * `n_lambda` (int): The number of wavelength samples between the edge wavelength and `lambda_min`.

* `"Tabulated"`: A tabulated continuum.
    * `unit`: sequence of two strings specifying the wavelength and cross-section units respectively (typically `["nm", "m^2"]`).
    * `value`: An sequence of sequences containing two float entries each: the wavelength of the sample, and the cross-section at that wavelength.

### Simplified format

* Only `"Tabulated"` continua are supported. All units in `"nm"` and `"m^2"`.
* Interpolation of cross-sections is implementation defined.

Collisional Rates
-----------------

The file will provide a top-level `collisions` sequence.
This sequence will contain the specification of the processes between each pair of levels of interest as a mapping.

### High-level format

The mapping describing collisional rates for each transition pair must contain:

* The key `transition` with value a sequence of two atomic level labels specifying the upper and lower levels of the transition by label.
* The key `data` with value a sequence of mappings describing the processes relating the two levels.

Each mapping in data must contain a field specifying the `type` (string).
The core types in CRTAF are:

| `type` | Description | Default Units |
|--------|-------------|---------------|
|"Omega" | Seaton's collision strength (excitation of ions by electrons) | Dimensionless |
|"CI"    | Collisional ionisation by electrons | "m3 s-1 K(-1/2)" |
|"CE"    | Collisional excitation of neutrals by electrons | "m3 s-1 K(-1/2)" |
|"CP"    | Collisional excitation by protons   | "m3 s-1" |
|"CH"    | Collisional excitation by neutral hydrogen   | "m3 s-1" |
|"ChargeExcH" | Charge exchange with neutral H (downward only) | "m3 s-1" |
|"ChargeExcP" | Charge exchange with protons (upward only) | "m3 s-1" |

All of these core types are rates which conventionally scale with temperature, so each process mapping will also contain:

* a mapping `temperature` containing a `unit` (string) and `value` (sequence of floats)
* a mapping `data` containing a `unit` (string) and `value` (sequence of floats) describing the rates.

### Low-level format

* All processes are provided with their default units.
* Interpolation of cross-sections is implementation defined.

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
Extensions are normally suggested and approved via the GitHub pull request system.