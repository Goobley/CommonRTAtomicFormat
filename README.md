Common Radiative Transfer Atomic Format
========================================


|   |   |   |   |
|---|---|---|---|
| __Maintainer__ | Chris Osborne | __Institution__ | University of Glasgow  |
| __License__ | ![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue) | | |


Description
-----------

This repository contains the specification for the Common Radiaitive Transfer Atomic Format (CRTAF).
CRTAF is intended as both a human readable and data interchange format for atomic data used in non-LTE radiative transfer codes.
This is achieved through the use of a clearly specified YAML document.
It defines both a high-level (intended for human interaction), and simplified formats (for simple integration into other codes with just a YAML parser).

CRTAF is units aware and supports a common set of functionality present in non-LTE radiative transfer codes such as [Lightweaver](https://github.com/Goobley/Lightweaver), RH, SNAPI.
The reference python package for interacting with CRTAF models is [`crtaf`](https://github.com/Goobley/crtaf-py).

This package is primarily of use to those working with atomic data to feed into radiative transfer models.

📖 Specification
----------------

The current specification is available [here](spec/core-spec.md). This is a living document, but will be versioned via  git tags, and archived via Zenodo.

Current extensions to the specification are available [here](spec/extensions/).


🤝 Contributing
---------------

We would love for you to get involved.
Please report any typos/bugs encountered on the GitHub issue tracker.

To request the addition of an extension/update the specification, please submit a pull request, or discuss in issues. It is expected that specification changes will also be accompanied by a tested set of changes/extension to the reference implementation [`crtaf`](https://github.com/Goobley/crtaf-py), but please discuss if you are not sure how to go about the desired changes.

We require all contributors to abide by the [code of conduct](CODE_OF_CONDUCT.md).

Acknowledgments
---------------

This format is based on the work of Tiago Pereira for [Muspel.jl](https://github.com/tiagopereira/Muspel.jl), along with the atoms as code approach of [Lightweaver](https://github.com/Lightweaver)
