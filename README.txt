CSS142P Modelling and Simulation - Final Project
Group 8: Ariola, Jon Luigi; Bauza, Xyrelle John; Caliliw, Marl Zeldrix; Jimenez, Justine Leee
Mapua University Makati parking entrance: discrete-event simulation

------------------------------------------------------------
1. WHAT THIS MODEL DOES
------------------------------------------------------------
Simulates one high-traffic operating day (07:00-19:00) at the campus parking
entrance. Based on group members' knowledge of the campus:
  - One wide lane serves both entering and exiting cars, with one guard on duty.
  - Only cars with a parking sticker may enter. The guard checks the sticker.
  - Cars without a sticker are refused, but they occupy the guard and the lane
    while being turned away.
  - Exiting cars drive out without a check, so they do not use the guard.
  - Parking is in a two-level basement.

Configurations compared:
  A  one guard, 07:00-19:00 (baseline)
  B  a second guard 07:00-09:00
  C  a second guard 07:00-10:00

Scenarios (30 replications each):
  - Arrival demand x0.8, x1.0, x1.2 (professor's instructions)
  - Processing time x0.8 and x1.2 at base demand (extra sensitivity test)
  - Stress test: demand x1.0 to x5.0 for A and B, to find how much traffic
    growth one guard can absorb before the line reaches the street

------------------------------------------------------------
2. HOW TO REPRODUCE THE RESULTS
------------------------------------------------------------
The executable model is CSS142P_Group8_Parking_Simulation.ipynb.

Option A: Google Colab (nothing to install)
  1. Open https://colab.research.google.com and choose File > Upload notebook
     (or the GitHub tab, and paste this repository's link).
  2. Choose Runtime > Run all.
  The first cell installs SimPy. If the notebook is opened without the data/
  folder, it creates the input files itself with the same estimates.

Option B: Jupyter on your own computer
  1. pip install -r requirements.txt
  2. Open the notebook from this folder and run all cells.

Either way it takes under a minute, and every file in results/ and figures/
is regenerated. All randomness derives from base_seed in
data/input_parameters.csv, so reruns give identical numbers.

Tested with Python 3.12, SimPy 4.1.2, NumPy 2.4, pandas 3.0, SciPy 1.17,
Matplotlib 3.10.

------------------------------------------------------------
3. FILES
------------------------------------------------------------
CSS142P_Group8_Parking_Simulation.ipynb   Model, verification, experiments,
                                          charts and findings
requirements.txt                 Python packages for running locally

data/arrival_rates.csv           Arrival rate (cars/hour) per 15-minute interval
data/input_parameters.csv        Processing times, sticker share, K, guard hours,
                                 replications, seed, with the basis for each value

results/replications.csv         One row per simulated day
results/summary_by_scenario.csv  Mean and 95% CI half-width for each measure
results/paired_differences.csv   B-A, C-A, C-B differences with 95% CIs
results/stress_test.csv          Demand x1.0 to x5.0, configurations A and B
results/queue_profile_base_demand.csv  Average line length per minute, 07:00-11:00
results/verification_mg1.csv     Verification against the M/G/1 formula

figures/queue_profile_base_demand.png
figures/peak_wait_by_demand.png
figures/stress_test.png

------------------------------------------------------------
4. INPUTS ARE ESTIMATES, NOT MEASUREMENTS
------------------------------------------------------------
Access to the entrance and its records was not available. The system's
structure (sticker-only entry, one guard, one lane, exits unchecked) comes from
group members' first-hand knowledge. The numbers below are estimates; the
sensitivity scenarios and the stress test show whether the conclusion depends
on them.

  Arrivals: about 214 cars 07:00-10:00, peaking at 100 cars/hour at 07:30,
    consistent with filling a two-level basement lot, then 25-30/hour at
    midday and 8-20/hour in the afternoon. About 400 entries per day.
  Sticker cars: 95% of arrivals; check time lognormal, mean 8 s, SD 4 s.
  Cars without stickers: 5%; turned away; lognormal, mean 45 s, SD 15 s.
  K (waiting spaces before the line reaches Pablo Ocampo Sr. Ext.): 8 cars.

To use better numbers, edit the two CSV files in data/ and rerun the notebook.
No code changes are needed.

------------------------------------------------------------
5. MODEL ASSUMPTIONS
------------------------------------------------------------
- Arrivals: non-homogeneous Poisson process, rate constant within each
  15-minute interval. Arrivals stop at 19:00; cars still in line are processed.
- One shared first-come, first-served line. With two guards, the wide lane lets
  two cars be checked side by side.
- Guard 2 takes no new car after going off duty but finishes the current one.
- Out of scope: finding a space inside the basement, exit traffic (it does not
  use the guard), balking and reneging.
- Each day starts with an empty line and idle guards.

------------------------------------------------------------
6. MEASURES
------------------------------------------------------------
avg_wait_sec        mean of (check start - arrival), all cars, in seconds
avg_wait_peak_sec   same, cars arriving 07:00-10:00 only
max_wait_sec        longest wait of the day
max_queue           largest number of cars waiting (excluding those being checked)
spillover_min       total minutes with at least K cars waiting
utilization         total busy guard-time / total guard-time on duty

------------------------------------------------------------
7. COMMON RANDOM NUMBERS AND VERIFICATION
------------------------------------------------------------
Every configuration and demand level in a replication uses the same random
streams: stream 0 for arrivals (by inverting the cumulative rate function),
stream 1 for sticker or no sticker, streams 2 and 3 for check times. Car i is
the same car in every scenario, so differences reflect staffing, not luck.

Verification (results/verification_mg1.csv): with one guard and a constant
arrival rate giving 75% utilization, the simulated mean wait matches the
Pollaczek-Khinchine M/G/1 formula, and the model's per-car waits match the
Lindley recursion exactly on the same inputs.
