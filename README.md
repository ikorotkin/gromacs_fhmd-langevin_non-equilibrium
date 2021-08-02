# gromacs_fhmd-langevin_non-equilibrium

Modified GROMACS with thermostat-consistent fully coupled Molecular Dynamics – generalised Fluctuating Hydrodynamics model for non-equilibrium flows

#

## Installation of the code

The installation procedure of the code is completely the same as installation of the original [Gromacs 2016](https://manual.gromacs.org/documentation/2016/install-guide/index.html).

The following is the typical installation in Linux System:

```bash
cd gromacs_fhmd-langevin_non-equilibrium
mkdir build
cd build
cmake .. -DGMX_BUILD_OWN_FFTW=ON -DCMAKE_INSTALL_PREFIX=$PWD
```

Here `$PWD` denotes the custom installation directory, for example, `/home/fhmd`.

```bash
make
make install
```

If you want to switch to debug mode, the `cmake` step above should be modified to:

```bash
cmake .. -DGMX_BUILD_OWN_FFTW=ON -DCMAKE_INSTALL_PREFIX=$PWD -DCMAKE_BUILD_TYPE=Debug
```

Providing an installation directory (instead of `$PWD`) is highly recommended. Quick and dirty installation may lead to subsequent installation to overwrite the previous version.

See [INSTALL](https://github.com/ikorotkin/gromacs_fhmd-langevin_non-equilibrium/blob/main/INSTALL) file for more details.

## Execution of the code

```bash
source /home/fhmd/bin/GMXRC  # (where /home/fhmd is the installation directory)
gmx grompp -f grompp.mdp -c conf.gro -p topol.top -o topol.tpr
gmx mdrun -s topol.tpr -ntomp 1
```

Option `-ntomp 1` means that MD-FH coupling currently does not support OpenMP (shared memory) parallelism. But MPI and Gromacs tMPI parallelisation is supported and will be activated by default.

## Parameters for the two-way coupling method

The parameters are listed in the directory named `FHMD_parameters`. Files `conf.gro`, `grompp.mdp` and `topol.top` are required by Gromacs for molecular dynamics simulation,
`coupling.prm` includes the parameters for the two-way coupling part. The following is a brief introduction of the main parameters (recommended). All parameters have been calibrated to stabilize the model.

```
scheme = 2          ; 0 - Pure MD, 1 - One-way coupling, 2 - Two-way coupling

S = -1              ; Parameter S (-1 - fixed sphere, -2 - moving sphere)

NF = 1              ; Non-equilibrium flow selection (0- Const velocity: Uflow_x=0 means Equilibrium state, 1-poiseuille)
Uflow_x = 0.05      ; maximum velocity in the x-direction for Non-equilibrium flow

R1   = 0.5          ; MD sphere radius for variable S, [0..1]
R2   = 1            ; FH sphere radius for variable S, [0..1]
Smin = 0            ; Minimum S for variable S
Smax = 0.5          ; Maximum S for variable S

alpha = 500         ; Alpha parameter for dx/dt and du/dt equations, ps^-1
beta  = 1000        ; Beta parameter for du/dt equation, ps^-1

eps_rho = 0.01      ; Eps_rho parameter (FH density fluctuations dissipator)
eps_mom = 0.01      ; Eps_mom parameter (FH momentum fluctuations dissipator)
eps_hy_rho = 0.5    ; Eps_hy_rho parameter (epsilon-hybrid scheme for rho' )
eps_hy_mom = 0.5    ; Eps_hy_rho parameter (epsilon-hybrid scheme for mom' )
eps_av_rho = 0.1    ; Eps_av_rho parameter (Artificial viscocity term dissipator for rho* )
eps_av_mom = 0.1    ; Eps_av_rho parameter (Artificial viscocity term dissipator for mom* )

tau = 0.01          ; tau parameter (Langevin-type thermostat), ps^-1

Nx = 9              ; Number of FH cells along X axis
Ny = 9              ; Number of FH cells along Y axis
Nz = 9              ; Number of FH cells along Z axis

NxMD = 5            ; Number of small-scale MD-FH cells along X axis
NyMD = 5            ; Number of small-scale MD-FH cells along Y axis
NzMD = 5            ; Number of small-scale MD-FH cells along Z axis

FH_EOS   = 1        ; EOS: 0 - Liquid Argon, 1 - SPC/E water
FH_step  = 10       ; FH time step dt_FH = FH_step * dt_MD
FH_equil = 0        ; Number of time steps for the FH model equilibration (for 1-way coupling)
FH_dens  = 602.181  ; FH mean density
FH_temp  = 298.15   ; FH mean temperature
FH_blend = 0.005    ; FH Blending: -1 - dynamic, or define static blending parameter (0..1)

Noutput  = 100      ; Write arrays to files every Noutput MD time steps (0 - do not write)
Flow_output  = 0    ; Starting step output flow information
```

## Comparison between MD-FH coupled simulation and pure MD simulation

A simple method to execute a pure MD simulation and simultaneously obtain the fluctuation parameters is to set `S = 0` in `coupling.prm`. Setting `scheme = 0` can also switch to pure MD simulations, but the fluctuations won't be output directly.

## Licensing

-   GROMACS is open source software distributed under the GNU Lesser General Public License (LGPL) Version 2.1 or (at your option) any later version. GROMACS includes optional code covered by several different licences. See [COPYING](https://github.com/ikorotkin/gromacs_fhmd-langevin_non-equilibrium/blob/main/COPYING) file for details.
-   AFM FH/MD add-on to GROMACS is fully open source under [MIT license](https://github.com/ikorotkin/gromacs_fhmd-langevin_non-equilibrium/blob/main/LICENSE_FH-MD).
