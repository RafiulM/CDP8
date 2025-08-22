# Project Requirements Document - CDP8

Project Requirements Document for CDP8 (Composers Desktop Project 8)

1. Project Overview and Objectives  
   1.1. Overview  
     – CDP8 is a suite of standalone, command-line audio processing tools (time-domain, spectral, spatial) and data utilities for computer music composition.  
     – The codebase comprises ~700 files in C/C++ organized under `dev/`, a CMake-based build system, third-party libraries (PortAudio, FFTW), and custom sound-file I/O modules.  

   1.2. Objectives  
     – Provide a stable, documented, cross-platform (Linux, macOS, Windows) build and release of CDP8 tools.  
     – Ensure each CLI tool correctly implements its audio-processing algorithm and file I/O.  
     – Improve maintainability: modularize code, reduce duplication, modernize build and memory-management practices.  
     – Maintain full compatibility with existing Sound Loom (`sloom`) integrations.  
     – Deliver comprehensive user and developer documentation, including samples and test suites.  

2. Functional Requirements  
   2.1. Core Audio-Processing Tools  
     – Spectral processes: phase-vocoder analysis/synthesis (`pvoc`, `specfilt`, `formants`), spectral blurring, convolution, morphing, granular synthesis.  
     – Time-domain processes: stretching (`stretch`), pitch-shifting (`repitch`), distortion, filtering, envelope extraction, modulation.  
     – Spatial audio: multi-channel panning, ambisonic decoding, delay-based reverberation.  
     – Synthesis: additive/spectral synthesis, wave-table synthesis (`synth`).  
   2.2. File I/O and Utilities  
     – Support standard sound formats (WAV, AIFF, AU) and CDP’s analysis formats (`.ana`, `.pvx`).  
     – Recording and playback via PortAudio tools (`recsf`, `pvplay`, `paplay`).  
     – Metadata and property manipulation (`sndinfo`, `pinfo`).  
     – Data table editing (`tabedit` tools) and breakpoint file handling (`brktopi`, `read_value_from_brktable`).  
   2.3. Command-Line Interface  
     – Consistent syntax and option parsing across all tools.  
     – Dynamic parameter control via command options, breakpoint tables, and Sound Loom protocol.  
     – Unified error-reporting conventions (negative return codes, global `errstr`).  
   2.4. Build and Packaging  
     – Cross-platform CMake scripts producing static/shared libraries and executables.  
     – Legacy `Makefiled.osx` support for macOS.  
     – Packaged releases including source, license, dependencies, and sample scripts.  

3. Non-functional Requirements  
   3.1. Performance  
     – Real-time or near-real-time processing for streaming tools (`pvplay`, `recsf`) using ring buffers and threading.  
     – FFT operations optimized via FFTW or custom rFFTW implementations.  
   3.2. Reliability and Stability  
     – Robust error checking for file I/O, memory allocation, parameter ranges.  
     – Automated test suites with ≥80% code coverage and continuous-integration builds.  
     – Zero known memory leaks (validated by Valgrind or similar).  
   3.3. Maintainability  
     – Consistent coding style: documented in a style guide.  
     – Modular design: extract common utilities (argument parsing, file I/O) into shared libraries (`cdplib`, `sfsys`, `portsf`).  
     – Reduce code duplication, especially in `main.c` files.  
   3.4. Portability and Compatibility  
     – Build and run on modern Linux distributions (glibc), macOS (Clang), Windows (MSVC/MinGW).  
     – Conditional compilation flags isolate platform-specific code.  
     – Backward compatibility with CDP7 data files and Sound Loom protocol.  

4. Technical Requirements  
   4.1. Languages and Standards  
     – C (C99 or later) for core DSP modules.  
     – C++ (C++11 or later) for FFTW wrappers and selective modules.  
     – Portable threading: pthreads on Unix, Win32 threads on Windows.  
   4.2. Build System  
     – CMake ≥3.10 with properly scoped `CMakeLists.txt` in each subdirectory.  
     – Option flags: `-DBUILD_SHARED_LIBS`, `-DUSE_SLOOM`, `-DENABLE_PORTAUDIO`.  
   4.3. Libraries and APIs  
     – FFTW3 or equivalent FFT library.  
     – PortAudio for real-time I/O.  
     – Custom sound-file system (`sfsys`), `portsf` for formatted file I/O.  
     – libaaio for non-blocking keyboard input.  
   4.4. Data Structures and Memory Management  
     – Central `dataptr` structure (`dz`) carrying all state; document each field.  
     – Explicit `malloc`/`free` with error checks; consider migrating to safer abstractions (e.g., `safe_alloc()`).  
   4.5. Documentation and Tooling  
     – Doxygen or Sphinx for API documentation.  
     – Sample scripts in `docs/`, plus `README.md`, `building.txt`, and process-specific notes.  

5. Dependencies and Prerequisites  
   5.1. Build-Time Dependencies  
     – C compiler (GCC, Clang, MSVC) supporting C99.  
     – CMake 3.10+.  
     – FFTW3 development headers and library.  
     – PortAudio development package.  
     – libaaio (bundled as `libaaio-0.3.1`).  
   5.2. Runtime Dependencies  
     – Dynamic libraries (if shared builds): FFTW3, PortAudio.  
     – For Sound Loom integration: Tcl/Tk headers and libraries (optional).  
     – Standard system libraries (pthread, m, stdc++).  
   5.3. Tools  
     – Valgrind (Linux) or similar for memory profiling.  
     – Continuous-integration service (Travis CI, GitHub Actions, AppVeyor).  
     – Code-coverage tool (gcov/lcov).  

6. Assumptions and Constraints  
   6.1. Assumptions  
     – Users are familiar with command-line interfaces and audio file conventions.  
     – Development team has C/C++ and DSP domain expertise.  
     – Existing audio algorithms are correct and require maintenance, not redesign.  
   6.2. Constraints  
     – Large legacy codebase with global state; refactoring must be incremental.  
     – Manual memory management must be preserved unless fully tested replacements are in place.  
     – Backward compatibility with user scripts and breakpoint tables.  
     – Limited unit-testability of some procedural modules without additional abstractions.  

7. Success Criteria  
   7.1. Build and Release  
     – Automated, reproducible build on Linux, macOS, Windows (CI badges green).  
     – Packaged source and binaries with versioned release notes.  
   7.2. Functional Validation  
     – All CLI tools execute sample workflows from `docs/current-notes.txt` without errors.  
     – Spectral and time-domain processing outputs match reference audio within defined tolerances.  
   7.3. Quality Metrics  
     – ≥80% code coverage by automated tests.  
     – Zero memory leaks or invalid memory accesses under Valgrind.  
     – No regressions in core functionality (verified by nightly regression tests).  
   7.4. Documentation and Adoption  
     – Up-to-date API and user documentation published online.  
     – At least one community-contributed tutorial or example added within three months of release.  

This document provides a structured roadmap for project managers and developers to stabilize, maintain, and evolve CDP8 while preserving its extensive audio-processing capabilities.

---
*Generated by Git Search API on 2025-08-22 00:39:31 UTC*
