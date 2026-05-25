This MATLAB code performs a probabilistic local stability analysis for a colluvial landslide under heavy rainfall. It uses Monte Carlo simulation to account for spatial variability of soil parameters and computes the Local Factor of Safety Margin (FLSM) along a predefined slip surface.

Overview
The code implements a modified infinite slope stability model with unsaturated soil mechanics (Montrasio & Valentino model) and a rainfall infiltration model (Green–Ampt type). A push–pull force equilibrium method (similar to Morgenstern–Price) is used to compute local safety factors for each slice. Monte Carlo sampling generates random realizations of material properties, and failure probabilities are derived from the distribution of FLSM.

Dependencies
MATLAB R2018b or later (tested with R2020a)

Statistics and Machine Learning Toolbox (for normrnd, lognrnd, ksdensity, ecdf)

Excel file reading capability (basic MATLAB supports xlsread, which works on Windows; for other OS consider readtable)

No additional third-party toolboxes are required.

Input Data Preparation
The code reads geometry data from an Excel file located at:

text
D:\文件\强降雨作用下破坏概率演化\案例\龙家台滑坡\龙家台.xlsx
You must change this path to match your local file.

Excel File Structure
The file should contain a sheet with 40 rows (one per slice) and 12 columns (A to L). The expected columns are:

Column	Description
A	X-coordinate of left top point (m)
B	Y-coordinate of left top point (m)
C	X-coordinate of right top point (m)
D	Y-coordinate of right top point (m)
E	X-coordinate of left bottom point (m)
F	Y-coordinate of left bottom point (m)
G	X-coordinate of right bottom point (m)
H	Y-coordinate of right bottom point (m)
I	(Unused in current code)
J	Y-coordinate of water table at left side (m)
K	(Unused)
L	Y-coordinate of water table at right side (m)
Important: The code assumes:

n = 40 slices (rows in the Excel file)

Columns are read as: X(j,1)...X(j,12) for j = 1..40

Geometry is defined in 2D profile (X horizontal, Y vertical)

Parameter Configuration
All parameters are defined in the Parameter Initialization section. You can modify them directly in the script.

Rainfall Parameters
q – rainfall intensity (m/s). Default: 4 mm/h → 4/3600/1000

t – rainfall duration (s). Default: 100 hours → 100*3600

Soil Properties (Statistical Moments)
The code generates random samples using normal or lognormal distributions. Default values are placeholders – you must replace them with your own site-specific data.

Variable	Distribution	Mean	Std Dev	Description
theta_delta	Normal	0.19	0.0019	Water content deficit (-)
phi	Lognormal	-1.2603	0.0799	Internal friction angle (rad, log‑space)
c	Lognormal	2.3929	0.09975	Effective cohesion (kPa)
s	Lognormal	4.7874	0.0100	Suction cohesion parameter (kPa)
psai	Lognormal	-0.2232	0.0100	Suction head at wetting front (m)
Ks	Lognormal	-13.9405	0.1980	Saturated hydraulic conductivity (m/s, log‑space)
gama_s	Normal	missing	missing	Saturated unit weight (kN/m³)
gama_a	Normal	missing	missing	Natural unit weight (kN/m³)
phi_b	Normal	missing	missing	Friction angle related to suction (rad)
Note: The current script references gama_s_mean, gama_s_std, gama_a_mean, gama_a_std, phi_b_mean but these variables are not defined. You must add them before the normrnd calls. Example:

matlab
gama_s_mean = 20.5; gama_s_std = 1.2;
gama_a_mean = 18.0; gama_a_std = 1.0;
phi_b_mean = 0.35;   phi_b_std = 0.05;
Also, the line phi_b = normrnd(phi_b_mean, phi_b, m1, n); contains a typo – it should be phi_b_std. Correct it.

Monte Carlo Settings
m1 – number of random samples (default 10,000)

n – number of slices (default 40, must match Excel rows)

Running the Code
Prepare your Excel file with the correct geometry.

Edit the script:

Change the xlsread file path.

Provide correct statistical moments for all soil parameters (especially gama_s, gama_a, phi_b).

Fix the typo in phi_b generation.

(Optional) Adjust rainfall parameters, Monte Carlo sample size.

Run the script in MATLAB:

matlab
>> run('code.m')
or click the Run button in the MATLAB editor.

The computation may take several minutes (depending on m1 and n). A progress message appears every 100 samples.

Outputs
After execution, the script produces:

1. Command Window Output
Progress messages

(No explicit final print, but you can add disp statements)

2. Figures
Figure 1: FLSM profile along the landslide

Subplot (a): Mean FLSM (±1 standard deviation) for each slice

Red dashed line at FLSM = 0 (stability limit)

Figure 2: Probability density (PDF) and cumulative distribution (CDF) for selected slices

Columns plotted: 2, 10, 18, 21, 29, 39 (you can change target_columns)

Blue curve: kernel density estimate (PDF)

Red curve: empirical CDF

3. Variables in Workspace
Important arrays saved in the workspace after run:

Variable	Size	Description
FLSM	[m1, n]	Local safety margin for each sample and slice. Negative = unstable.
FLSM_mean	[1, n]	Mean FLSM per slice
FLSM_std	[1, n]	Standard deviation of FLSM per slice
failure_prob	[1, n]	Probability of failure (FLSM < 0) per slice
quantile_matrix	[5, n]	Quantiles (0.01, 0.25, 0.5, 0.75, 0.99) of FLSM per slice
You can access them in the workspace after the script finishes.

Notes and Troubleshooting
Common Issues & Fixes
Issue	Probable Cause	Solution
Undefined function or variable 'gama_s_mean'	Missing parameters for saturated/natural unit weight and suction friction angle.	Add the missing mean/std variables before normrnd.
Error using xlsread	File path not found.	Update the path to your own Excel file.
Matrix dimensions must agree in phi_b line	Typo: phi_b used instead of phi_b_std.	Replace phi_b with phi_b_std.
Very slow execution	Large m1 or n.	Reduce m1 to 1000 for testing.
NaN or infinite FLSM	Division by zero or unrealistic geometry.	Check geometry: ensure l(j) > 0, alpha(j) is valid, and water table h within slice height.
Code Improvements Suggested
Replace xlsread with readtable for cross‑platform compatibility.

Vectorize inner loops for speed (optional, but current loop structure is clear).

Add input validation and error handling.

Save results to .mat or Excel automatically.

Output Interpretation
FLSM < 0 → local instability (failure) at that slice.

Probability of failure (e.g., 0.35) means 35% of Monte Carlo realizations show FLSM < 0 for that slice.

Mean FLSM should be > 0 for overall stable sections; values near zero indicate marginal stability.

License & Contact
This code is provided as‑is for research and educational purposes. No warranty is implied. For questions or modifications, please contact the original author (details in the script header, if any).

Last updated: 2025 (based on script content)
