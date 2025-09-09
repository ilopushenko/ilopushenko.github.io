---
layout: post
date: 2025-09-09 14:45:00+0200
inline: true
related_posts: false
---

Today I would like to share something which could have been called "yet another Mie theory solver". Indeed, there are so many solvers implementing Mie scattering that practically anyone can compute e.g. scattering efficiencies without putting much effort into seacrhing. Yet sometimes, there is a need to ensure that the result is accurate enough, or to compute just something which specific code does not provide, or to simply answer the question: "Which code is better for my needs?". <br><br>

For accurate results, a well-tested MIEV0 program by Dr. Warren J. Wiscombe [has been available for many years](https://www.researchgate.net/publication/253485579_Mie_Scattering_Calculations_Advances_in_Technique_and_Fast_Vector-speed_Computer_Codes). It is written in Fortran, has really comprehensive documentation and freely available sources, and can truly be called a reference implementation of the Mie theory. Somehow, in my research activities I have always missed a tool which would be as fast and reliable as MIEV0, and easy to use at the same time. Well, at least as easy as possible for embedding in some other projects, for validation of other scattering models, or just for fast and accurate estimates. <br><br>

So here it is: [a MATLAB interface to W. Wiscombe's MIEV0 program](https://doi.org/10.5281/zenodo.17069741)! This is a MATLAB MEX library compiled from MIEV0 sources supplied with Fortran interface between MIEV0 and MATLAB, which had to be implemented in order to get this thing working. I have successfully tested the built binary file against original tests authored by Dr. Wiscombe, so this is a ready-to-use tool suitable for the beginners and for experienced scientists, and I hope that it really helps any of your research. For details, please see the [GitHub repo](https://github.com/ilopushenko/miev0_matlab_interface). And for appropriate reference, please have a look at the open access paper [Wiscombe, W. "Improved Mie scattering algorithms," Appl. Opt. 19, 1505-1509 (1980)](https://doi.org/10.1364/AO.19.001505).