## Statistical software for spatial biology

*__Cardinalverse__* is a Github organization for software and research in spatial molecular biology led by Prof. Kylie Ariel Bemis in collaboration with Olga Vitek Lab at the Khoury College of Computer Sciences at Northeastern University.

The organization develops and maintains a collection of statistical software packages for spatial omics in the *__Cardinalverse__* ecosystem started by the *Cardinal* package. Originally released on Bioconductor in 2015 and winner of the 2015 John M. Chambers Statistical Software Award by the American Statistical Association (ASA), *Cardinal* provides analytical workflows for mass spectrometry imaging (MSI) experiments.

## R Packages

We are currently in the process of refactoring *Cardinal* into an ecosystem of related packages. The primary goal is a stronger and more intuitive separation of responsibilities and dependencies compared to the current monolithic approach where all pre-processing, visualization, and statistical learning methods live in the *Cardinal* package. A secondary goal is to reduce the dependency (and tight coupling) with the backend *matter* package (which is currently used for various processing and iteration tasks in addition to its original stated purpose of handling out-of-memory arrays), which should make new *Cardinal* code contributions more atomic and accessible.

The proposed design for the *__Cardinalverse__* ecosystem is visualized in the graph below:

```mermaid
flowchart BT
    CardinalIO --> Cardinal
    CardinalCore --> Cardinal
    CardinalCore --> CardinalStats
    CardinalStats --> Cardinal
    Cardinal --> CardinalOmics
    CardinalStats --> CardinalOmics
```

#### Cardinal

*Cardinal* is the frontend for MSI-based workflows. It should be a pure R package for easy user installation from source (for providing release previews and hotfixes without requiring compilation).

It implements the classes most users will manipulate directly, including:

- `MSImagingArrays` : unaligned mass spectra with pixel metadata
- `MSImagingExperiment` : aligned mass spectra with pixel metadata and feature metadata
- `PositionDataFrame` : data frame with required position and run metadata
- `MassDataFrame` : data frame with required m/z metadata
- `SpectraArrays` : spectra array lists or matrices

It depends on *CardinalIO* for I/O, *CardinalCore* for spectral processing and visualization, and *CardinalStats* for statistical learning.

It should avoid dependencies on any packages other than those above and core Bioconductor infrastructure such as *S4Vectors*. It should avoid unnecessary dependencies on extended Bioconductor infrastructure that is better handled by *CardinalOmics*.

#### CardinalIO

*CardinalIO* is responsible for reading and writing file formats directly supported by the *Cardinalverse* (i.e., those we implement and maintain ourselves rather than depend on third-party packages or utilities). This is primarily **imzML**. Support for most other formats (such as **HDF5**, **zarr**, **SpatialData**, etc.) should use third-party packages for support.

It includes compiled C/C++ code for efficiently parsing XML (via the vendored *pugixml* header-only C++ library).

It should avoid dependencies on any packages other than *matter*, *ontologyIndex*, and core Bioconductor infrastructure.

#### CardinalCore

*CardinalCore* is responsible for low-level processing and visualization tools. These tools are the building blocks of *Cardinal*'s processing and visualization. Many of these low-level tools (e.g., `knnsearch()` and `findpeaks()`) have been implemented in *matter* to remove compiled C/C++ code from *Cardinal*. However, these tools really belong in a *Cardinalverse* package, so that *matter* can return to its original scope of domain-agonistic out-of-memory data structures.

Examples of appropriate functionality for *CardinalCore* include low-level implementations of:

- Normalization
- Smoothing
- Baseline removal
- Peak detection and alignment
- Contrast enhancement for images
- Nearest neighbor searches
- Range searches
- Visualization

It may include compiled C/C++ for these tasks. Any *Cardinalverse* functionality that may be tightly coupled with C/C++ code (but is __not__ directly related to I/O or statistical modeling or machine learning) should be implemented here instead of *Cardinal*. All functions implemented here should operate on basic R data structures (e.g., atomics, lists, and data frames) rather than *Cardinal* data structures.

It should avoid dependencies on any packages other than *matter*, *ggplot2*, and core Bioconductor infrastructure.

#### CardinalStats

*CardinalStats* is responsible for implementation of statistical modeling and machine learning for the *Cardinalverse* ecosystem. These implementations are the building blocks of the analytical workflows in *Cardinal*. Many of these low-level functions (e.g., `nscentroids()` and `sgmix()`) have been implemented in *matter* to remove compiled C/C++ code from *Cardinal*. However, these tools really belong in a *Cardinalverse* package, so that *matter* can return to its original scope of domain-agonistic out-of-memory data structures.

Examples of appropriate functionality for *CardinalStats* include low-level implementations of:

- Summary statistics
- Dimension reduction
- Segmentation/clustering
- Classification
- Regression
- Statistical testing
- Cross-validation

It may include compiled C/C++ for these functions. Any *Cardinalverse* functionality directly implementing statistical modeling or machine learning *algorithms* should be implemented here instead of *Cardinal*. All functions implemented here should operate on basic R data structures (e.g., atomics, lists, and data frames) rather than *Cardinal* data structures.

It depends on *CardinalCore* and other R packages for implementing specialized statistical models such as *lme4*, *lmerTest*, etc.

It should avoid dependencies without a history of sustained maintenance. It should avoid dependencies with system requirements.

#### CardinalOmics

*CardinalOmics* will provide orchestration of multimodal experiments that integrate MSI data.

It may depend on other R packages in the broader Bioconductor ecosystem to support spatial multi-omics. Any *Cardinalverse* workflows on multimodal experiments should be implemented here, including co-registration and methods for applying *CardinalStats* analyses to non-*Cardinal* data structures such as `SummarizedExperiment` and `SpatialExperiment`, etc.

#### matter

The *matter* package (that currently serves as the computing backend for *Cardinal*) is notably missing from the above design. A major goal of refactoring *Cardinal* is to simplify *matter* into a replaceable data backend.

Simply stated: a *Cardinal* pipeline that never needs out-of-memory data processing should have no dependency on *matter* for its functionality. We will continue to rely on *matter* as a default backend for out-of-memory data processing, but it should be fully replaceable with other backends such as a `DelayedMatrix`, `HDF5Matrix`, or ordinary `matrix`.

<!--

**Here are some ideas to get you started:**

🙋‍♀️ A short introduction - what is your organization all about?
🌈 Contribution guidelines - how can the community get involved?
👩‍💻 Useful resources - where can the community find your docs? Is there anything else the community should know?
🍿 Fun facts - what does your team eat for breakfast?
🧙 Remember, you can do mighty things with the power of [Markdown](https://docs.github.com/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
-->
