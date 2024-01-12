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
* It may contain a multi-line string of notes from the model author.

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
This table will contain the element `symbol` (a string), and the `atomic_mass` (in amu).
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

* An `energy` table containing a `value` for the energy (float), and the unit used `string`
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

* energy is in typical spectroscopic units "cm^-1"

Spectral Lines
--------------

### High-level format

### Simplified format


Continua
--------

Collisional Rates
-----------------

