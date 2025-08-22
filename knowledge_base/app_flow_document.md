# App Flow Document - CDP8

Application Flow Document for CDP8  
=================================

1. High-level Architecture Overview
-----------------------------------
CDP8 is a collection of command-line audio processing tools built in C/C++. Each tool is structured into four logical layers:

•  Presentation Layer (CLI)  
   – main.c in each tool  
   – Command-line parsing, help/usage output  
•  Parameter & Configuration Layer  
   – ap_<tool>.c files define legal parameters, variants, break-point files  
   – Break-point subsystem for time-varying parameters  
•  Processing Engine Layer  
   – <tool>.c files implement core DSP algorithms (time-domain or spectral)  
   – Shared DSP utilities in include/ (e.g. FFT wrappers, windowing, filtering)  
   – Central dataptr (“dz”) structure carries buffers, file handles, parameters  
•  I/O & Support Layer  
   – sfsys/ and portsf libraries manage audio file I/O (WAV, AIFF, PVX, ANA)  
   – PortAudio for real-time playback/recording (pvplay, recsf)  
   – External DSP libs (FFTW, reverb library)  

At build time, CMake orchestrates compilation of externals, utility libraries, and each tool into standalone executables.

2. Component Interaction Diagram (text-based)
----------------------------------------------
  
   [User Shell]  
        │  
        ▼  
   [Executable main.c]  
        │─ parses CLI args ─►[Parameter Manager (ap_*.c)]  
        │                           │  
        │                           ▼  
        │                     [dataptr “dz”]  
        │       (populates file/I/O props, flags, param arrays)  
        │                           │  
        │                           ▼  
        │                     [DSP Core (<tool>.c)]  
        │         (reads dz.inBuffer, param tables; writes dz.outBuffer)  
        │                           │  
        │                           ▼  
        │                     [I/O Layer (sfsys/portsf)]  
        │         (read_samps, write_samps, sndopenEx, sndcreat)  
        │                           │  
        ▼                           ▼  
   [Input Audio File]           [Output Audio File]  
        ▲                           │  
        │                           ▼  
   [Disk / Std-in]            [Disk / Std-out]  

3. Data Flow Description
------------------------
1. User invokes a tool (`cdpblur infile.wav outfile.wav -param1 val1 …`).  
2. main.c  
   • Calls init routines (signal handlers, error buffers).  
   • Invokes command-line parser (`get_tk_cmdline_word` or sloom parser).  
3. Parameter Manager (ap_blur.c)  
   • Defines `aplptr` parameter specs: count, types, ranges, default values, breakpoint support.  
   • Validates parameters (`check_param_validity_and_consistency`).  
   • Populates `dataptr dz`:  
     – dz.infile, dz.outfile strings  
     – dz.param[] arrays, dz.bpTables pointers  
     – dz.format flags (sample rate, channels)  
4. I/O Layer  
   • Opens input file via `sndopenEx(&dz.inSF, dz.infile, SFM_READ, &props);`  
   • Allocates inBuffer = malloc(frames × channels × sizeof(float))  
   • Reads frames in blocks: `read_samps(&dz, blockSize)` into inBuffer  
5. DSP Core (blur.c)  
   • For each block:  
     – Optionally convert to spectral domain (FFT)  
     – Apply blur algorithm (convolution with gaussian spectral window)  
     – Inverse FFT if used  
     – Apply breakpoint-driven param updates mid-stream  
   • Writes result into outBuffer  
6. I/O Layer  
   • Creates output file via `sndcreat_formatted(&dz.outSF, dz.outfile, &props);`  
   • Writes outBuffer blocks: `write_samps(&dz)`  
7. Cleanup & Exit  
   • Free buffers, close files  
   • Return 0 on success or negative error code  

4. User Journey / Workflow
---------------------------
1. Clone Repository:  
     git clone https://github.com/ComposersDesktop/CDP8.git  
2. Build:  
     mkdir build && cd build  
     cmake ..  
     make  
   – Produces binaries like `blur`, `distort`, `env`, `pvplay`, etc., in bin/  
3. Inspect Usage:  
     ./blur -h  
     ./pvplay --help  
4. Typical Run:  
     ./pvoc infile.wav pvf.ana 1024 256 HANN   # spectral analysis  
     ./repitch pvf.ana out.wav -f1 0.5 -f2 2.0  # apply time-varying repitch  
5. Optionally chain tools in shell scripts or Sound Loom GUI  

5. “API Endpoints” and Interactions
-----------------------------------
Though standalone, each tool offers a CLI “endpoint” mapping to internal functions:

Tool       | CLI Command     | Entry point     | Setup Function              | Core DSP Function  
-----------|-----------------|-----------------|------------------------------|--------------------  
blur       | blur            | main() in main.c| setup_blur_application(aplptr)| do_blur(dz*)  
distort    | distort         | main()          | setup_distort_application    | do_distortion  
env        | enve            | main()          | setup_envel_application      | do_envel  
pvoc (analyze) | pvoc          | main()          | setup_pvoc_application       | do_pvoc_analysis  
pvplay     | pvplay          | main()          | (no ap_, uses pvplay.c)      | pv_playback_thread  
recsf      | recsf           | main()          | recsf_setup                  | recsf_record_thread  

Internal “API calls” within a tool:  
•  applib functions: set_legal_param_structure2(), set_legal_option_and_variant_structure2()  
•  File I/O: sndopenEx(), fgetfbufEx(), sndcreat_formatted(), write_samps()  
•  DSP utilities: fft_real(), ifft_real(), window_apply(), convolve(), filter_process()  

6. Database Schema Overview
----------------------------
CDP8 does not use a relational database. Persistent data is stored in:  
•  Audio files: WAV/AIFF headers carry metadata (sample rate, bit depth, channels)  
•  Analysis files (`.ana`, `.pvx`): CDP’s custom binary header + float frames  
•  Break-point tables: plain text files with two columns (time, value)  

Key in-memory data structures (defined in structures.h):  
Struct SFPROPS {  
    int format;           // encoding format code  
    long samplerate;      // sample rate in Hz  
    int channels;         // channel count  
    double duration;      // in seconds  
};  
Struct dataptr {  
    char *infile, *outfile;  
    SFPROPS inSF, outSF;  
    float *inBuffer, *outBuffer;  
    int *iparam;          // integer parameters  
    float *param;         // floating parameters  
    BreakPointTable **bpTables;  
    int processID, mode;  
    char errstr[256];  
    …  
};  

7. External Service Integrations
--------------------------------
•  PortAudio (externals/paprogs)  
     – Real-time playback/recording, ring-buffer threads in pvplay.c, recsf.c  
•  FFTW (fastconv, mxfftd.c)  
     – FFT wrappers for spectral transforms  
•  Custom reverb library (externals/reverb)  
     – Allpass & comb filters, nested Moorer reverb  
•  libaaio (libaaio/)  
     – Non-blocking keyboard input for interactive shells  
•  sfsys & portsf (externals/portsf & sfsys/)  
     – Multi-format sound-file reading/writing  

8. Error Handling and Fallbacks
-------------------------------
•  Functions return int: 0 on success, negative codes on error  
•  Errors accumulate in dataptr.errstr via sprintf()  
•  On fatal error:  
     – do_error() prints errstr, jumps to cleanup  
     – print_messages_and_close_sndfiles() closes files gracefully  
•  Parameter parsing fallback: if optional parameters missing, defaults from ap_<tool>.c are used  
•  Break-point file missing: subsystem issues warning, uses static parameter value  
•  Memory alloc failure: code tests for NULL, writes “Out of memory” to errstr, and aborts  
•  I/O failure: sndopenEx and write_samps report file open/write errors, abort pipeline  

Summary
-------
CDP8’s control flow begins with a CLI entry in main.c, flows into parameter setup via ap_*.c, routes data and params into the central dataptr structure, then into the core DSP routines in <tool>.c, and finally through the I/O layer to read/write audio. Error handling is uniform across modules using negative codes and a shared errstr buffer. Integration with external libraries (PortAudio, FFTW, reverb) is abstracted behind well-defined I/O and DSP utility interfaces.

---
*Generated by Git Search API on 2025-08-22 00:39:31 UTC*
