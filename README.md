MetamerObserverModel
====================

Summary
-------
This codebase implements the image "metamer" model from Freeman & Simoncelli (2011) and related tooling used to analyze images, compute local higher-order texture statistics in overlapping pooling regions, synthesize images that match those statistics (metamers), and evaluate model performance on psychophysical-style tasks (e.g. 2AFC simulations).

High-level capabilities
----------------------
- Analyze images with a complex steerable pyramid to compute local statistics (pixel moments, band magnitudes, autocorrelations, cross-correlations, etc.). See `functions/metamerAnalysis_w.m`.
- Synthesize images (metamers) that match those local statistics. See the `functions/metamerSynth/` subfolders (synthesis, analysis, helper, main).
- Run simulations and compute model performance on 2-interval forced-choice (2AFC) tasks and other evaluation scripts. See `functions/getPerf2afc.m` and related scripts in `functions/` and `simulations/`.
- Utility scripts for generating pooling windows that tile the image or tile polar/log-eccentricity (to mimic receptive fields): `create_windows.m` / `create_windows.sh`.
- Scripts to estimate model parameters for target or synthesized images: `estimate_params_textureModel_target.m`, `estimate_params_textureModel_synth.m`, and related shell wrappers.
- Includes convenience scripts for downloading example data: `download_data.sh`.

Where code lives (important folders)
-----------------------------------
- `functions/` — core MATLAB functions used by the model (analysis, synthesis, plotting, helpers).
  - `metamerSynth/` — synthesis/analysis pipeline used to create metamers (contains `readme.txt` with more details).
  - `matlabPyrTools/` — included steerable/steerable-pyramid helper code used for pyramidal decompositions (required by analysis/synthesis).
- `simulations/` — simulation outputs and helper scripts used by the performance evaluation pipeline.
- `mask/`, `textureSynth/` and other subfolders contain masks, texture synthesis helpers, and additional tooling.
- Top-level helper scripts: `create_windows.*`, `download_data.sh`, `estimate_performance_textureModel.sh`, etc.

Dependencies
------------
- MATLAB (the project is written in MATLAB scripts and assumes a MATLAB environment).
- `matlabPyrTools` (steerable/complex steerable pyramid routines). A copy of this toolbox is included under `functions/matlabPyrTools`, but you may also use a system-installed version if preferred.

Key entry points / How to run (quick)
------------------------------------
1. In MATLAB, add this repository to your path. For example:

```matlab
addpath(genpath('/path/to/SteerablePyramids/external/MetamerObserverModel'))
```

2. Create pooling windows (if needed) and inspect examples:

```matlab
% generate default windows
create_windows;

% run example analysis & synthesis (see functions/metamerSynth/main for scripts)
% Example: compute statistics for an image and synthesize a metamer
params = metamerAnalysis(oim, m, opts);
% then run synthesis pipeline in metamerSynth
```

3. Run performance evaluations:

```matlab
% prepare responses and parameters, then
[sampPerf_L1, sampPerf_L2, sampPerf_L4, out] = getPerf2afc(allResp, inParams, iP);
```

For more complete examples and workflows, inspect the scripts in `functions/metamerSynth/main/` and the shell scripts at the top level.

Texture model parameter and performance scripts
-----------------------------------------------
`estimate_params_textureModel_target.m` and `estimate_params_textureModel_synth.m` compute the same kind of masked texture-model parameter vector, but they run over different image sets.

`estimate_params_textureModel_target.m` analyzes a single original target image. In the default script, it reads `target_images/llama.png`, loads the matching `2048 x 2048` pooling/window file, center-crops the image, runs `metamerAnalysis`, collects and masks duplicate parameters with `collectParams` and `collectParamMask`, and saves one `params_<image>_s=<scaling>_a=<aspect>.mat` file back into the target-image folder.

`estimate_params_textureModel_synth.m` analyzes synthesized/metamer images. It searches under `metamers/<metamer_type>/<model>_model/<image>/<scaling>/`, finds `.png` files, skips any image that already has a matching `params_*.mat`, loads the window file once when needed, runs the same analysis and masking pipeline, and saves a separate parameter file next to each synthesized image. The script defines both `metamers_energy_ref` and `metamers_energy_met`, but its current loops are written as `for mer = 1% : length(met_types)` and `for s = 1% : length(scalings)`, so it only processes the first metamer type and first scaling directory unless those comments are removed.

`estimate_performance_textureModel.sh` is a SLURM batch wrapper for running MATLAB 2AFC performance simulations from already-computed parameter files. It launches an array job over 20 named images, takes one command-line argument for the analysis-window scaling, hardcodes `scenario = 'bar'`, and reads from scratch-space folders such as `target_images_bar`, `metamers_bar`, and `results_bar`. For each image, it loads target parameters, finds metamer parameters from `metamers_energy_met` and `metamers_energy_ref`, and runs both `tar_vs_met` and `met_vs_met` comparisons with `getPerf2afc`.

The performance script simulates a noisy observer over `2500` trials for each condition and writes `resultsTextureModel.csv` containing fields such as `met_scaling`, `mean_L1`, `mean_L2`, `mean_L4`, `noise`, `image`, `type`, and `window_scaling`. It is intended to be run with `sbatch estimate_performance_textureModel.sh <window_scaling>` after target and metamer parameter files have already been generated. Two small caveats: the script uses shell arrays despite declaring `#!/bin/sh`, so it expects Bash-like behavior, and the comment says "2 jobs" even though `#SBATCH -a 0-19` creates 20 array jobs.

Background / references
-----------------------
This code implements the metamer model described in:

Freeman, J. & Simoncelli, E. P. (2011). Metamers of the ventral stream. Nature Neuroscience, 14(9).

The synthesis approach and many of the underlying ideas are related to the Portilla & Simoncelli (2001) parametric texture model.

Notes & Caveats
----------------
- Designed and developed between ~2009–2013; the MATLAB code style and compatibility may assume older MATLAB versions. Some scripts use shell wrappers (`.sh`) for batch processing.
- There is no explicit project-wide license file in this copy; consult the upstream repository for license details if you need to redistribute or modify the code.
- The code expects reasonably large images (e.g., 512x512) for the default parameters.

Where to look next
------------------
- `functions/metamerSynth/readme.txt` — package-specific readme about metamers (contains condensed explanation).
- `functions/metamerAnalysis_w.m` — detailed comments showing how local statistics are computed.
- `functions/getPerf2afc.m` — shows how model responses are turned into simulated psychophysical performance.

If you want, I can also:
- Commit the new README to the local cloned repo and optionally push to a fork on GitHub (if you provide credentials or set up remote), or
- Expand the README with example commands tailored to your MATLAB setup.
