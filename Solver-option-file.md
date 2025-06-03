Solver options for mix99s
================
2025-06-03

Some of the solver options can be specified from the command line. The
easiest way to execute solver is to give option `-s`which uses default
values in solving breeding values, and produces standard output files:

`mix99s -s`

More options can be specified in a solver option file that the solver
reads from the standard input:

`mix99s < solver_option_file.slv`

NOTE: When BOTH the solver option file AND the command line options are
given, only the command line options are used by default. With command
line option `-i` this can be changed so that the solver option file is
read AND the options from the command line override the corresponding
solver option file values.

## Solver option file

<!-- ![Example of solver option file .slv](RR-milk_SOF.png) -->

Example of a solver option file:

    # RAM: RAM demand: H=high, M=medium, L=low
        H
    # STOP: Maximum_number_of_iterations, Stopping_criterion, Criterion (A/R/D)
        2000           1.0e-06                  R              f
    # RESID: Calculate residuals? (Y/N)
        N
    # VALID: N=no, P=prediction, S=sum of effects, Y=YD, D=DYD, I=IDD, G=generate
        N
    # VAROPT: (N)o HV, (S)tart HV, (C)ontinue HV, (F)inale, (E)stimation VC by EM
        N
    # SOLTYP: Solution files? (N)o, (Y)es, (A)itken, (H)alf-Chebychev
        Y

The following tabs introduce the options for each line in solver option
file. Default or recommended values and options are **bolded**.

### RAM

A line with at least one character defining the use of random access
memory:

    # RAM: RAM demand: H=high, M=medium, L=low
        H

The use of RAM:

- **`H`** = high. All required vectors are kept in memory. This option
  is recommended.
- `M` = medium. Solutions are stored on the disk.
- `L` = low. Solutions and residuals are stored on the disk.
- `X` = eXtra. Like H, but some extra in parallel computing (parallel
  solver only).

Additional options that can be given after the definition of RAM use:

- `NO/YES` = checking of release information.
- `nt n` = number CPU in multi-threading is set to be `n` (integer).
- `srm m` = sparse regress matrix read to memory for matrix number `m`
  (integer?).
- `sp v`= second level preconditioner of value `v` for regress matrix.
- `noc` = no residual covariances used. Assumes none were given (single
  processor solver only).
- `noQ` = no genetic groups for $A_{gg}^{-1}$ term, e.g. no
  $Q´A_{gg}^{-1}Q$ in single-step models having $A_{gg}^{-1}$ through
  the PEDIGREE option.
- `IOP/IM/CHM/PAR` = in single-step, product of $A_{gg}^{-1}$ times
  vector can be performed using one of four alternative approaches.
  Approach IOP uses iteration on pedigree, IM uses iteration in memory,
  but CHM uses CHOLMOD and PAR uses MKL PARDISO library.
- `MEL` = in single-step, reads $G^{-1}$ or $T$ matrix from file to
  memory. The MEL option uses efficient matrix multiplication during PCG
  iteration.
- `MEM` = is like option `MEL`, but slower and uses slightly less
  memory, when $T$ matrix is used. Same as MEL when $G^{-1}$.
- `MES` = does not read $G^{-1}$ or $T$ to memory. Memory efficient, but
  slow.
- `MEB b` = is like option `MEL`, but makes the computations in blocks
  of size b in ssGTBLUP. Can be faster than MEL for very large
  matrices.  
- `RDS` = instructs to use a small block size when calculating
  contributions form regression design matrices. Currently implemented
  in single processor solver only.
- `RDM` = similar to `RDS`, but uses medium block size and keeps SNP
  marker matrices byte-packed in memory. Currently in single processor
  solver only.
- `RDL` = similar to `RDM`, but uses large block size and keeps SNP
  marker matrices byte-packed in memory. Currently in single processor
  solver only.
- `RDX` = instructs to keep regression design matrices fully in (double
  precision real) memory. Currently in single processor solver only.
- `RDB b` = similar to `RDL`, but uses given block size abs(`b`)
  (absolute value of `b`). If `b` is positive, keeps SNP marker matrices
  byte-packed in memory. Currently in single processor solver only.
- `RDU m` = simmilar to `RDB` but instructs to use given amount of
  memory (`m`) when calculating contributions of regression design
  matrices. Full regression design matrices are kept in memory if
  possible or at least byte-packed SNP matrices if possible. Block size
  is also calculated from the given memory limit. Memory size `m` is
  given as <amount><unit> where <amount> is an integer and optional
  <unit> is one of K (kilo), M (mega), G (giga, default), T (tera), and
  P (petabytes). Example: RDU 12G . Currently in single processor solver
  only.
- $\omega$ = value for the $\omega$ multiplier of matrix $A_{gg}^{-1}$.
  In practice, value of $\omega$ is typically about $0.6 - 0.8$.

### STOP

A line with four entries for setting up the PCG iteration:

    # STOP: Maximum_number_of_iterations, Stopping_criterion, Criterion (A/R/D), enforce (F)
        2000           1.0e-6                 R                F

**PCG (preconditioned conjugate gradient) iteration information**

Maximum number of iterations (integer), e.g.:

- `2000`
- **`5000`**

Stopping criterion (real), e.g.:

- **`1.0e-4`**
- `5.0e-5`

Convergence indicator to which the stopping criterion is applied:

- `A` = CA. Relative difference (residual) between right-hand and
  left-hand side of the MME considering all equations of the additive
  genetic animal (individual) effects only.
- `R` = CR. Relative difference between right-hand and left-hand side of
  the MME considering all equations.
- `M` = CM. Relative difference between preconditioned right-hand and
  left-hand sides of the MME considering all equations.
- **`D`** = CD. Relative difference between solutions of the last two
  iteration rounds. Note that if CD is chosen as the convergence
  indicator, the stopping criterion must be met by two consecutive
  iterations.

NOTE: If the confergence indicator `CD` is used, a suitable stopping
criterion for the majority of analyses would be `1.0e-4`. For some
multiple trait models and for estimation of variance components it was
found that the criterion should be stricter.

Enforcing:

- `F` = The solver will consider the STOP option line only in case an
  enforcing character `F` is specified for the fourth entry. Otherwise,
  default values will be used.

### RESID

A line with one character specifying the calculation of residuals:

    # RESID: Calculate residuals? (Y/N)
        N

Calculate residuals:

- `Y` = Yes. Residuals will be written into the file `eHat.data0` when
  using the single processor solver. The order of the residuals
  corresponds with the order of observations in the input data file. In
  the case of parallel processing, each process writes an own
  `eHat.data(i)` file with the process number `(i)` at the end of the
  file name. Then, the order of the residual files correspond to the
  order of observations in the input data file beginning with file
  zero (0) up to number of processes minus one.
- `N` = No. No residuals are written.
- `h` = This option is only available in parallel solver and will create
  the file(s) ARsiwi.data(i), which contain information about the
  heterogeneity of variance in the residuals. These files are needed
  only when accounting for heterogeneous variance.

### VALID

A line with one entry, which instructs the solver to calculate for each
observation a corresponding, here specified, sub-quantity of the applied
model line, or to instruct the solver to simulate observations based on
the specified model:

    # VALID: N=no, P=prediction, S=sum of effects, Y=YD, D=DYD, I=IDD, G=generate
        N

Options:

- `N` = None. None of the options are requested.
- `P` = Predictions. For each observation, the predicted value
  ($\hat{y}$) is written to the file(s) `yHat.data(i)`.
- `S` = Selected Model Factor’s Sum. For each observation the sum of
  selected model factors is written to the file(s) `sHat.data(i)`. The
  selected factors must be specified on a following line.
- `Y` = Yield Deviations (YD). For each observation the corresponding YD
  will be written to the file(s) named `YD.data(i)`. The factors
  included into the YD must be specified on a following line.
- `I` = Individual Daughter Deviations (IDD). For each observation the
  corresponding IDD will be written into the files(s) named
  `IDD.data(i)`. The factors included in the IDD must be specified on a
  following line.
- `D` = Daughter Yield Deviations (DyD). The solver will calculate for
  each observation the corresponding IDD and will use it for the
  calculation of DYDs baesd on the approach of Mrode and Swanson (2004).
  For this option the calculated DYD will be written to a formatted file
  named `Soldyd`. The factors included into the DYD must be specified on
  a following line.  
- `G` = Generate observations. This option is available in the single
  processor solver only. The solver will not solve the model, but
  instead will generate for each observation in the data a simulated
  observation ($\tilde{y}$). Therefore, for all effects in the model
  true solutions will be simulated based on the provided variance
  components. Fixed effect solutions will be set to zero. The true
  solutions are written to the standard solution files. The generated
  observations will be written into the file named `ySim.data0`. This
  file can be used in a future run to replace real observation by
  simulated observations. When specifying `G` a `SEED` option line must
  be included after the`VAROPT` option line.
- `R` = Deregression.

The options `Y`, `I`, `D` and `G` are not supported when solving
non-linear models.

The options `S`, `Y`, `I`, or `D` will require adding of a second line,
which specifies which factors of the model are included into the
calculation of the specified quantity.

One line with as many integers as there are factor columns defined in
the REGRESS instruction line. This is equal to the first integer value
of the REGRESS instruction line. The order of the integer values on the
FACTOR line corresponds to the order of the factors specified on the
REGRESS instruction line.

    # VALID: N=no, P=prediction, S=sum of effects, Y=YD, D=DYD, I=IDD, G=generate
        S
        # FACTOR: (0/1)
        1 1 0 0 

Is the corresponding factor of the model included into the calculation
of the desired quantity:

- `1` = The factor will be included into the specified quantity.
- `0` = The factor will be excluded from the specified quantity.

### VAROPT

A line with one entry that specifies different options related to the
adjustment for heterogeneous variance or to the estimation of variance
components:

    # VAROPT: (N)o HV, (S)tart HV, (C)ontinue HV, (F)inale, (E)stimation VC by EM
        N

Options:

- `N` = None. None of the options are requested.
- `E <f n>` = Estimation of variance components. The option `E` will
  instruct solver to estimate variance components by a stochastic Monte
  Carlo Expectation Maximization REML (MC EM REML) algorithm.
  - A special case is option `EI` for estimation of variance components
    of a MACE model.
  - An additional (optional) instruction can be given after the `E` (or
    `EI`) character. This instruction has two entries; the character `f`
    and an integer number `n`. The optional instructions are needed in
    case certain variance component parameters are meant to be fixed.
- `S` = Start-up cycle for heterogeneous variance adjustment. After the
  pre-processor has performed a maximum number of 20 iterations
  (specified on the STOP option line) it will write heterogeneity of
  variance estimates to files named `SiWi.data(i)`. These files will be
  used by mix99hv to create the input data files for the applied
  variance model that describes the heterogeneity of variance in the
  data.
- `C`= Cycle between models for solving the multiplicative mixed model.
  The option is needed for the heterogeneous variance adjustment and
  will instruct the pre-processor to discontinue in certain intervals
  the iteration process and make system calls for solving the variance
  model by a second, simultaneous analysis. The process will continue
  until both models have converged.

#### `VAROPT`: `E` or `EI` (Variance Component Estimation)

**Variance Component Estimation**

In case `E` or `EI` is defined on the `VAROPT` line, three additional
instruction lines have to be given:

    # VAROPT: (N)o HV, (S)tart HV, (C)ontinue HV, (F)inale, (E)stimation VC by EM
        E
        # STOPE maximum number of EM steps, samples/step, convergence crit.
          1000                                2             1e-07
        # SEED random number generator
          R
        # PRE-PROCESSOR
          /home/bin

`STOPE`, a line with three (or four) entries:

Maximum number of MC EM REML rounds (integer), e.g.:

- **`1000`**
- `-1000` = A negative sign indicates that previously run REML
  estimation is to be continued using this new total number of rounds.

Number of data samples generated and analyzed within a REML round
(integer), e.g.:

- `2`
- **`5`**

Stopping criterion for the REML analysis (real), e.g. :

- `1.0e-8`
- **`1.0e-9`**

Stopping criterion for the PCG iteration of the sampled data
(real)(optional), e.g.:

- **`1.0e-4`** = The same as for the real data on option line `STOP`.
  \[Check this\]

`SEED`, a line with one entry defining the type of the seed used by the
random number generator for generating the data samples:

- **`D`** = Default initialization by call to random_seed.
- `R` = The random number generator is initialized based on the system
  clock.
- `G` = The user can specify the seeds for the ransom number generator.
  If option `G` is specified, *j* integers must be provided in the next
  line.

`PRE-PROCESSOR` path to the directory where the pre-processor executable
is located:

- `/home/bin` = Pre-processor is located in the bin.
- `""` or `-` = Empty directory name indicates that the pre-processor is
  assumed to be located in a directory that is included in the search
  PATH.

#### `VAROPT`: `S` or `C` (Heterogeneous Variance Adjustment)

**Heterogeneous Variance Adjustment**

In case `S` or `C` is defined, an additional line needs to be specified
with as many integers as there are factor columns defined in the REGRESS
instruction line. This is equal to the first integer value of the
REGRESS instruction line. The order of the integer values on the
`ADJUST` line corresponds to the order of the factors specified on the
REGRESS instruction line.

    # VAROPT: (N)o HV, (S)tart HV, (C)ontinue HV, (F)inale, (E)stimation VC by EM
        S
        # ADJUST (0/1)
        1 1 1 0 

Is the corresponding factor of the model included into the adjustment of
heterogeneous variance:

- `1` = The factor will be included into the HV adjustment.
- `0` = The factor will be excluded from the HV adjustment.

### SOLTYP/TYPSOL

A line with one character, which specifies the way solutions are
handled:

    # SOLTYP: Solution files? (N)o, (Y)es, (A)itken, (H)alf-Chebychev
        Y

- `Y` = Yes, give standard solution files. Solution files are written in
  text format.
- `N` = No. No solutions are written. This is useful when specifying
  option `S` on the `VAROPT` option line.
- `D` = DMUINPformat. Option will produce binary and ASCII files with
  the pseudo data. This option is needed only for a simultaneous
  estimation of variance components for the non-linear Gompertz function
  model by, for example, using the DMU package for the variance
  component estimation.

One of the two options (`A`, `H`) must be defined when using MiX99 for
solving the variance model for adjustment of heterogeneous variance. The
option will instruct the pre-processor to accelerate the solutions of
the variance model between consecutive heterogeneous variance adjustment
cycles. The solutions are written to binary files (SolfixB for strata of
the first effect and SolaniB for strata of the second effect in the
variance model).

- `A` = Accelerated solutions using Aitken acceleration. This option is
  suitable, if convergence is dominated by a single large eigenvalue.
- `H` = Accelerated solutions using a Half-Chebychev golden ratio
  procedure (Hesterberg, 2005). This step-lengthening method follows a
  golden ratio procedure. The option is robust and therefore recommend
  for routine evaluations.

NOTE: Solving a standard linear model expects `Y`. All other options
(`N`,`A`,`H`) are related to adjustment of heterogeneous variance or to
non-linear Gompertz models.
