# Muon Analysis Notebook Walkthrough

This document summarizes the steps in the notebook and what each cell does.

## Overview
The notebook processes muon detector data from text files in the `Data/` folder. It parses pulse timing columns, filters out events with channel-1 activity, computes time differences ($\Delta t$) between two channel-0 pulses, visualizes histograms, and demonstrates a simple cutflow workflow for event selection.

## Step 1: Imports and Setup
- Enables inline plotting in the notebook.
- Imports core utilities and scientific libraries: `Path` (file paths), `numpy`, `matplotlib`, `pandas`, `re` (regex), and `math`.
- Imports `display` for showing figures in notebook outputs.

## Step 2: Parse Channel-0 Time Differences
Defines `read_ch0_deltas_from_file(path)`:
- Opens a data file and scans for a header line that starts with `Ch0_time1` (with or without units).
- Normalizes the header by stripping unit parentheses and splits it into column names.
- Locates column indices for:
  - `Ch0_time1` and `Ch0_time2`
  - any channel-1 time columns
- For each data row:
  - Parses numeric values.
  - Skips the row if any channel-1 time column is positive (indicates Ch1 activity).
  - Keeps the row if both `Ch0_time1` and `Ch0_time2` are positive.
- If no header is found, falls back to legacy column indices (conservative heuristic).
- Builds a `pandas.DataFrame` and adds `delta_t = Ch0_time2 - Ch0_time1`.

## Step 3: Parse Channel-0 Start Times
Defines `read_times_from_file(path)`:
- Similar header detection as above.
- Collects `Ch0_time1` values for rows without channel-1 activity.
- Returns a NumPy array of `Ch0_time1` values.

## Step 4: Plot Per-File $\Delta t$ Histograms
Defines `plot_ch0_delta_per_file(data_dir='Data', bins=100, save=False, show=True, max_display=3)`:
- Scans the data directory for `.txt` files.
- For each file:
  - Reads `delta_t` values using `read_ch0_deltas_from_file`.
  - Prints statistics: count, min, max, mean.
  - Plots a histogram of $\Delta t$.
  - Optionally saves the plot to disk.
  - Displays only up to `max_display` figures inline to avoid large output.

## Step 5: Run the Per-File Histograms and Print Counts
Executes the histogram routine:
- Calls `plot_ch0_delta_per_file('Data', bins=200, save=False, show=True, max_display=3)`.

Then for each data file:
- Reads `Ch0_time1` values.
- Prints total detections and a binned histogram table (bin start/end and count).

## Step 6: OR File Histogram Clipped to 20 microseconds
- Loads the OR file: `Data/MUO temp filtered pulse data OR.txt`.
- Computes $\Delta t$ values.
- Clips to $\Delta t \le 20\,\mu s$ (`tmax = 20e-6`).
- Plots a histogram restricted to that window.

## Step 7: Cutflow Utility Functions
Defines reusable functions for DataFrame-based event selection:

- `cut_low_voltage(df, threshold1=0.5, threshold2=0.5)`
  - Removes rows where `Ch0_amp1` or `Ch0_amp2` is below the thresholds.

- `cut_time_window(df, dt_min=None, dt_max=None)`
  - Ensures `delta_t` exists (computes it if possible), then filters to a time window.

- `subtract_noise(df, bins=50, delta_t_bkg_only=20e-6, dt_min=None, dt_max=None)`
  - Estimates a flat background rate from the tail region ($\Delta t > 20\,\mu s$).
  - Builds a histogram and subtracts expected background counts.
  - Returns background-subtracted counts, edges, and the estimated background rate.

- `build_cutflow(df, cuts)`
  - Applies a sequence of cuts and records event counts and efficiencies.

- `write_cutflow_table(cutflow_df, path)`
  - Writes the cutflow table to CSV.

## Step 8: Example Cutflow Run
- Loads the OR file into a DataFrame (`df_raw`).
- Defines two cuts:
  - `low_voltage` with thresholds 0.1 and 0.01
  - `time_window` with $1.5\,\mu s \le \Delta t \le 12\,\mu s$
- Runs `build_cutflow` and displays the table.
- Saves the cutflow to `cutflow_table.csv`.

## Step 9: Final Histogram After Cuts
- Takes the post-cut DataFrame.
- Ensures `delta_t` exists.
- Plots a histogram of the final $\Delta t$ distribution.

## Step 10: Semi-log Plot of Raw Counts
- Uses `df_raw` (uncut raw data) and ensures `delta_t` exists.
- Builds a histogram over $0$ to $25\,\mu s$.
- Fits a straight line to $\log(D)$ in the $10$ to $20\,\mu s$ background window.
- Plots $\log(D)$ vs $dt$ with the fitted background line.

## Step 11: Semi-log Plot of $D_2$ After Cuts
- Uses `df_after` (final cuts) and ensures `delta_t` exists.
- Estimates the background model from the $10$ to $20\,\mu s$ window and subtracts it to form $D_2$.
- Propagates uncertainties to $\log(D_2)$ using $\sigma_{D_2} \approx \sqrt{D + B}$.
- Performs a weighted linear fit to $\log(D_2)$ over $1.5$ to $12\,\mu s$.
- Computes an effective lifetime $\tau = -1/m$ and $\chi^2/\mathrm{dof}$.
- Plots $\log(D_2)$ with error bars and lines for $\tau$ and $\tau \pm 1\sigma$, normalized to the same total counts in the fit window.

## Outputs Produced
- Inline figures for per-file histograms (limited display).
- Printed detection counts and histogram tables for `Ch0_time1`.
- A clipped histogram for the OR file ($\Delta t \le 20\,\mu s$).
- A CSV cutflow table written to `cutflow_table.csv`.
- A semi-log $\log(D)$ plot with a background fit.
- A semi-log $\log(D_2)$ plot with fit, error bars, and lifetime lines.
