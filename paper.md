---
title: 'Lumabi: a Python package to streamline the computation of phonon-resolved luminescence spectra of defects in solids'
tags:
  - Python
  - materials science
  - defect physics
  - luminescence
  - density functional theory
  - vibronic peaks
authors:
  - name: Julien Bouquiaux
    orcid: 0000-0003-1982-052X
    corresponding: true
    affiliation: 1
  - name: Matteo Giantomassi
    orcid: 0000-0002-7007-9813
    affiliation: 1
  - name: Samuel Poncé
    orcid: 0000-0003-1159-8389
    affiliation: 1
  - name: Yongchao Jia
    orcid: 0000-0002-7319-9546
    affiliation: 2
  - name: Masayoshi Mikami
    orcid: 0000-0002-5598-4882
    affiliation: 3
  - name: Xavier Gonze
    orcid: 0000-0002-8377-6829
    affiliation: 1

affiliations:
 - name:  Institute of Condensed Matter and Nanosciences, Université catholique de Louvain, B-1348 Louvain-la-Neuve, Belgium
   index: 1
 - name: Yanshan University, Hebei Key Laboratory of Applied Chemistry, Yanshan University, 066004 Qinhuangdao, P. R. China
   index: 2
 - name: Materials Design Laboratory, Science & Innovation Center, Mitsubishi Chemical Corporation, Yokohama 227-8502, Japan
   index: 3 
  
date: 22 April 2025
bibliography: paper.bib

---

# Summary

![Lumabi logo](Lumabi_logo.pdf){width=50%}

Lumabi is a Python package integrated within the Abipy framework [@gonze2020abinit] designed to automate and streamline the computation of phonon-resolved luminescence spectra of defects in inorganic solids using the ABINIT first-principles software application [@gonze2002first;@gonze2009abinit;@gonze2016recent;@gonze2020abinit]. The package addresses the growing need for efficient, reproducible workflows in materials science [@lejaeghere2016reproducibility;@bosoni2024verify], particularly in the study of defect-related luminescent properties, which are critical for applications ranging from quantum technologies [@wolfowicz2021quantum;@dreyer2018first] to down-conversion phosphors materials used in white light LEDs [@pust2015revolution;@lin2017inorganic;@fang2022evolutionary]. Lumabi automates key steps in the computational workflow, from initial $\Delta$SCF density-functional theory calculations with constrained occupations, to the generation of defect phonons mode in large supercells, right through the final generation of luminescence spectra based on the Huang-Rhys theory [@huang1950theory;@jin2021photoluminescence]. Tutorials and examples, written in the form of Jupyter books, can be found at [https://jbouquiaux.github.io/lumi_book/intro.html](https://jbouquiaux.github.io/lumi_book/intro.html).

# Statement of need

The study of defect-induced luminescence in materials is crucial for understanding and designing materials with specific optical properties. However, the computational workflow required to accurately predict these properties is complex and often labor-intensive, involving multiple-stage density-functional theory (DFT) calculations, phonon computations, and post-processing.

Given such importance of defects in materials and the complexity of the involved workflow to simulate them, a number of software packages have been developed to manage the pre- or post-processing of general defect calculations [@naik2018coffee;@pean2017presentation;@goyal2017computational;@broberg2018pycdt;@kumagai2021insights;@neilson2022defap;@arrigoni2021spinney;@shen2024pymatgen]. The few that focus on luminescent properties [@Kavanagh2024;@turiansky2021nonrad;@cavignac2024] are all interfaced with the commercial software application VASP [@kresse1996efficiency], and only provide the post-processing part of DFT computations following the formalism proposed by Alkauskas et al. [@alkauskas2014]. Also, the generation of defect phonon modes in large supercells following an embedding of the interatomic force constants is currently not available in any software application to our knowledge.  

Divided in four main Python modules, Lumabi provides a solution to automate all the necessary tasks to compute phonon-resolved luminescence spectra of defects. The DFT computations are performed with the open-source ABINIT software application [@gonze2002first;@gonze2009abinit;@gonze2016recent;@gonze2020abinit]. The generation of ABINIT input files, the automatic workflow management, and the post-proccessing work are done with the help of the Abipy package, which is interfaced with Pymatgen [@ong2013python]. The code is interfaced with the Phonopy [@togo2015first;@togo2023first] package for the phonon-related tasks. This tool is suited for researchers looking for accurate simulations of defect luminescence with limited human intervention. Moreover, the systematic generation of well-structured data through this automated workflow could enable data-driven searches for new phosphors [@hariyani2023guide] and could provide a robust foundation for training machine-learning models [@lee2025machine] to accelerate the discovery of materials with targeted luminescent properties. 

# Software Description, Features, and Computational Workflow

The code is structured around four main Python modules, each handling a different stage of the computational workflow. While these modules are designed to be used seamlessly, the output of one providing the input of the next, the individual building blocks can be used independently with ease. We describe here the overall working principles of each block. For a more practical approach, it is suggested to follow the [online tutorials](https://jbouquiaux.github.io/lumi_book/intro.html) of the different modules and related examples.  

## LumiWork Module 

![The LumiWork module, an AbiPy Workflow that automates ABINIT DFT tasks with $\Delta$SCF constrained occupations.](LumiWork.pdf)

A computational workflow to compute phonon resolved PL spectra of defect system typically begins with the LumiWork module, which automates ABINIT DFT tasks with $\Delta$SCF constrained occupations [@jones1989density;@hellman2004potential]. Users input the defect supercell structure (e.g. a cif file), DFT parameters (e.g., plane-wave kinetic energy cutoff, stopping SCF criterion,...), and constrained occupations of the Kohn-Sham states that aim at mimicking the excited state of the system under study. This module manages two structure relaxations, for both the ground and excited states and offers optional single-shot static SCF and non-SCF band structure calculations. Note that the workflow is dynamic: the relaxed excited state is not known in advance, which requires the creation of inputs at run-time.

The `LumiWork` class inherits from the `Work` Abipy class. Hence, it benefits from the error detection and automatic restarts handling available for all ABINIT-related Works (such as an approaching time limit constrained by a job manager, or an unconverged task). This ensures that most of the computations proceed smoothly without interruptions. The important outputs from this module are in netcdf format, which serve as the basis for subsequent modules.

## $\Delta$SCF Post-Processing Module 

![The $\Delta$SCF module, designed to post-process $\Delta$SCF constrained-occupation calculations using a one-dimensional configuration-coordinate model.](dSCF_post_process.pdf)

The next step in the workflow is handled by the $\Delta$SCF post-processing module. This tool takes the netcdf output files from the previous LumiWork module and processes them following a one-dimensional configuration-coordinate model (see references [@jia2017first;@bouquiaux2021importance]). This analysis provides detailed insights into the luminescence characteristics of the defect under study, computing properties such as transition energies, Huang-Rhys factors, effective phonon frequencies, and lineshapes following this 1D model or within a semi-classical approximation. It is also able to help in the analysis of the atomic relaxation (generating for instance automatic VESTA [@momma2011vesta] files with relaxation vectors).

In the case of experimental lineshapes with resolved phonon peaks, it is important to note that the lineshapes obtained from this tool *are not* supposed to present a good agreement with experimental lineshapes at the level of the phonon peaks. Indeed, the model only considers a single effective phonon frequency $\omega_{\rm{eff}}$, and one should use the next modules to obtain a better agreement. Still, the 1D model is able to give the global spectrum shape, or the total Huang-Rhys factor, which can be compared to experiment.

## IFCs Embedding Module 

![The IFCs embedding module, allowing to calculate defect phonons in large supercells.](IFCs_embedding.pdf)

The Interatomic Force Constants (IFCs) Embedding module has the role to calculate defect phonons in large supercells at the $\mathbf{q}=(0,0,0)$ ($\Gamma$ point) wave vector. This module addresses the computational challenges associated with capturing both the coupling with long-wavelength phonons and the localized phonon modes introduced by defects. Direct methods, such as Density Functional Perturbation Theory (DFPT) [@gonze1997dynamical] and finite difference approaches [@togo2015first], are impractical for large supercells due to their high computational costs. First employed in the context of the luminescence of the NV center by Alkauskas et al. [@alkauskas2014], the embedding approach has been then used in various other systems [@jin2021photoluminescence;@bouquiaux2023first;@razinkovas2021vibrational;@maciaszek2023application;@jin2022vibrationally]. The procedure relies on the short-range nature of interatomic forces to build IFCs matrices for defect supercells containing thousands of atoms, targeting the dilute limit.

First, the real-space interatomic force constants for the defect system $C_{\kappa\alpha,\kappa'\alpha'}^{\mathrm{defect}}$ are computed within a relatively small supercell. Typically, this is achieved using a finite-difference approach at the $\Gamma$ point using Phonopy.

Second, the IFCs of the pristine system $C_{\kappa\alpha,\kappa'\alpha'}^{\mathrm{pristine}}$ are computed in a much larger supercell. A practical approach involves using DFPT on the unit cell to calculate the dynamical matrix across a $\mathbf{q}$-mesh. This is often followed by Fourier interpolation to obtain a denser $\mathbf{q}$-mesh, equivalent to a large supercell at the $\Gamma$ point. Such folding procedure that yields $C_{\kappa\alpha,\kappa'\alpha}^{\mathrm{pristine}}$ has been implemented, and receives as input an ABINIT DDB file (or a phonopy object) and produces as output a Phonopy object containing the folded pristine phonons.

Third, an embedded IFC matrix $C_{\kappa\alpha,\kappa'\alpha'}^{\mathrm{emb}}$ is constructed using the following rules: If both atoms $\kappa$ and $\kappa'$ are within a sphere centered around the defect, with a cut-off radius $R_c$, then $C_{\kappa\alpha,\kappa'\alpha'}^{\mathrm{emb}}$ is set to $C_{\kappa\alpha,\kappa'\alpha'}^{\mathrm{defect}}$. For all other atomic pairs, the embedded IFC is set to the pristine value: $C_{\kappa\alpha,\kappa'\alpha'}^{\mathrm{emb}} = C_{\kappa\alpha,\kappa'\alpha'}^{\mathrm{pristine}}$. The cut-off radius $R_c$ is typically determined by the size of the initial supercell used to compute the defect IFCs. Additionally, one can use a second cut-off radii $R_b$. If two atoms are not within a sphere of radii $R_b$, the IFC is set to zero. This allows one to obtain a sparse matrix, for which faster diagonalization techniques may be used, although the current implementation does not natively support such technique. Finally, the Acoustic Sum Rule is reinforced, following the conventions of reference [@jin2021photoluminescence].

The code performs a mapping between the structures of the pristine and defect computations. Such mapping is not trivial because the atomic positions of the defect supercell might not be the same as the pristine positions. Also, vacancies or interstitials should be handled with care during the mapping. For this reason, the mapping is done in a semi-automatic way, meaning that the user is responsible for the input of the different structure manipulations needed to go from the pristine cell to the defect cell. For details about this mapping, several examples are provided with the code. In the end, a Phonopy object containing the embedded IFCs and the new large defect supercell structure is created. For the treatment of the Born effective charges (BECs), if the BEC of the substituted atom was computed in the defect phonon calculation, it is replaced by the BEC of the dopant atom. If not, one can choose to use the most common oxidation state of the dopant atom as its new BEC. The same approach applies to interstitials, but in this case, the dopant atom and its corresponding BEC are simply added. For vacancies, the atom and its BEC are removed.

## Lineshape Calculation Module 

![The lineshape module, allowing to compute the temperature-dependent spectra.](lineshape.pdf)

As a final step, the Lineshape module is used. The main task of this module is to compute the Huang-Rhys spectral function $S(\hbar\omega)$ [@alkauskas2014; @bouquiaux2023first], and to generate temperature-dependent photoluminescence (PL) spectra using the efficient generating function approach [@jin2021photoluminescence].

The code expects as inputs the zero-phonon line energy, the atomic displacements $\Delta R_{\kappa\alpha}$ or forces $\Delta F_{\kappa\alpha}$ induced by the electronic transition (conveniently obtained from the $\Delta$SCF post-process block), and the phonon modes as a Phonopy object (potentially obtained from the IFCs embedding module). Notice that the use of the displacements is only compatible if the phonon supercell is of the same size as the $\Delta$SCF supercell. The use of the forces (equivalent under the harmonic approximation) allows one to use efficiently the previous block and enlarge the supercell size, ensuring a proper convergence of the Huang-Rhys spectral function [@alkauskas2014;@jin2021photoluminescence;@bouquiaux2023first]. An analysis of the different phonon mode localization can also be performed.

# Examples and Applications

This computational workflow has been widely used for inorganic phosphors activated with Eu$^{2+}$ defects and has also been tested on a variety of other systems, such as the F-center (oxygen vacancy) in CaO and the NV center in diamond. Its versatility allows it to be applied, in principle, to any kind of point defect.
These developments have been particularly useful in understanding the luminescence properties of technologically significant red-emitting Eu-doped phosphor materials. Notably, the workflow has been applied to SrAl$_2$Li$_2$O$_2$N$_2$:Eu$^{2+}$ and SrAlLi$_3$N$_4$:Eu$^{2+}$, shedding new light on their phonon sideband [@bouquiaux2021importance;@bouquiaux2023first]. We refer the reader to the accompanying notebook tutorials for practical examples demonstrating the application of this workflow.


# Acknowledgements

TO FILL 

# References
