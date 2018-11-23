# Preface

C++ Best Practices: A Forkable Coding Standards Document

This document is meant to be a collaborative discussion of the best practices in C++. It complements books such as *Effective C++* (Meyers) and *C++ Coding Standards* (Alexandrescu, Sutter). We fill in some of the lower level details that they don't discuss and provide specific stylistic recommendations while also discussing how to ensure overall code quality.

In all cases brevity and succinctness is preferred. Examples are preferred for making the case for why one option is preferred over another. If necessary, words will be used.


<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/Text" property="dct:title" rel="dct:type">C++ Best Practices</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="http://cppbestpractices.com" property="cc:attributionName" rel="cc:attributionURL">Jason Turner</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.

*Disclaimer*

This is a condensed version of Jason Turner's cppbestpractices. I removed the rules less relevant for our purposes and added 
some points specific to real time audio processing. We (meaning the MHA team) do not currently follow all or even most of 
these rules. Some of our workflows are mostly legitimized by the combination of historic reasons and inertia.
I however would strongly suggest to use these rules when starting a new project, even if it is only a new plugin 
within MHA.
