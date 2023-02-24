# Table of Contents

- [Table of Contents](#table-of-contents)
- [Installation](#installation)
  - [OceanWave3D repository (without Harwell)](#oceanwave3d-repository-without-harwell)
  - [2. Lapack, blas, tmglib](#2-lapack-blas-tmglib)
  - [3. Sparsekit](#3-sparsekit)
- [Files](#files)
  - [4. Input files](#4-input-files)
  - [5. Output files](#5-output-files)
- [Input File Formats](#input-file-formats)
  - [6. Main input file (Oceanwave3D.inp)](#6-main-input-file-oceanwave3dinp)
  - [6. OceanWave3D.init file](#6-oceanwave3dinit-file)
  - [6. Bathymetry file (fname_bottom)](#6-bathymetry-file-fname_bottom)
- [Output File Formats](#output-file-formats)
  - [Random wave coefficients file (eta0_coeffs)](#random-wave-coefficients-file-eta0_coeffs)
  - [Wave elevation file at inlet (eta0_irregular)](#wave-elevation-file-at-inlet-eta0_irregular)
  - [Fort.N files](#fortn-files)

# Installation

## OceanWave3D repository (without Harwell)

https://github.com/boTerpPaulsen/OceanWave3D-Fortran90

## 2. Lapack, blas, tmglib

https://github.com/Reference-LAPACK/lapack

## 3. Sparsekit

wget http://www-users.cs.umn.edu/~saad/software/SPARSKIT/SPARSKIT2.tar.gz

<br>
<br>

# Files

## 4. Input files

The list of input files are:

| FILEIP(N) | Description                                                                                                       |
| --------- | ----------------------------------------------------------------------------------------------------------------- |
| 1         | Main input file, default: "Oceanwave3D.inp". <br>The filename can be specified as the first command line argument |
| 2         |                                                                                                                   |
| 3         | Initial condition file (if it exists) "OceanWave3D.init"                                                          |
| 4         | Bathymetry file (if it exists in fname_bottom)                                                                    |

<br>
<br>

## 5. Output files

The list of input files are:

| FILEOP(N) | Description                                                                                                   |
| --------- | ------------------------------------------------------------------------------------------------------------- |
| 1-16      | Log files (numbers 8->23)                                                                                     |
| 12        | "local.smoothing" written to by "LocalSmoothing2D"                                                            |
| 13        | "relax.chk" written to by "PreprocessRelazationZones"                                                         |
| 14        | "roller.dat" written to by "detect_breaking"                                                                  |
| 15        | "Pdamp.chk" written to by "PreprocessPDampingZones                                                            |
| 16        | "bathymetry.chk" written to by OceanWave3DT0Setup                                                             |
| 21        | "eta_coeffs" written by "random_wave_coefficients"<br> "eta0_irregular" written by "random_wave_coefficients" |

<br>
<br>

# Input File Formats

## 6. Main input file (Oceanwave3D.inp)

The main input file is read by `ReadInputFileParameters.f90`. The filename is stored in the `GlobalVariables`-module as a 40-character string `filenameINPUT`, and is assigned the default name `OceanWave3D.inp` in `Initialize.f90`.

| Line number        | Variable                                                                                                                                                                                                                                                   | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1                  | HEAD(1)                                                                                                                                                                                                                                                    | Header line                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| 2                  | IC<br>.<br>.<br>.<br>.<br>.<br>IncWaveType<br>.<br>.<br>.<br>accel_tol_fact<br>.<br>.                                                                                                                                                                      | initial condition<br> <0 =Read initial conditions from file `OceanWave3D.init`<br>0=defined by `funPressureTerm.f90`<br>1=NL standing wave<br>2=shoaling on a smooth beach<br>3=Whalin bar <br> (opt) incident wave-type<br>0=none<br>1=stream function<br>2=linear irregular waves <br> (opt) local filtering downward vertical acceleration limit in m/s2. <br> Theorectical breaking should occur between 0.5 and 1.0 <br> Default value is 1000 i.e. no smoothing                                                                                                                                                                                                                                                                                                                    |
| 3                  | Lx<br>Ly<br>Lz<br>FineGrid%Nx<br>FineGrid%Ny<br>FineGrid%Nz<br>GridX<br>GridY<br>GridZ<br>.<br>.<br>GhostGridX<br>GhostGridY<br>GhostGridZ<br>.<br>.<br> fname_bottom                                                                                      | Length of domain in the x-direction <br>Length of the domain in the y-direction <br>Length of domain in the z-direction <br>Number of grid nodes in the x-direction<br>Number of grid nodes in the y-direction<br>Number of grid-nodes in the z-direction<br>Type of grid in the x-direction (0/1=even/clustering)<br>Type of grid in the y-direction (0/1=even/clustering)<br>Type of grid in the z-direction<br>0 = Even node-distribution in z-dir<br>1 = Uneven node distribution in z-dir <br>Ghostgrid off/on (0/1) in the x-direction <br>Ghostgrid off/on (0/1) in the y-direction<br>Ghostgrid off/on in the z-direction<br>0=Kinematic condition will be imposed directly<br>1=Ghost point layer included below bottom<br> (opt) filename with bathymetry definition for Lz<=0 |
| 4                  | alpha<br>beta<br>gamma<br>alphaprecond<br>betaprecond<br>gammaprecond                                                                                                                                                                                      | alpha<br>beta<br>gamma<br>alphaprecond<br>betaprecond<br>gammaprecond                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| 5                  | Nsteps<br>dt<br>timemethod<br>.<br>.<br>CFL<br>extrapolationONOFF<br>.<br>time0                                                                                                                                                                            | Number of time steps<br>dt<br>Time-integration method<br>1=Classical fourth-order Runge-Kutta (RK4)<br>2=Carpenter & Kennedy low storage five-stage RK (RK5)<br>CFL constant given by user<br>1=Optimization of RK scheme using extrapolation on seperate RK stages will be employed.<br>Starting time for this run (opt, default=0)                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| 6                  | g<br>rho<br>.                                                                                                                                                                                                                                              | Gravitational acceleration<br> (opt) Water density <br> Default value is 1000 kg/m^3                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| 7                  | solver<br>.<br>.<br>Precond<br>.<br>.<br>.<br>.<br>MGCoarseningStrategy<br>.<br>GMRESmaxiterations<br>reltol<br>abstol<br>maxit<br>cyclet<br>nu(1)<br>nu(2)<br>MGmaxgrids                                                                                  | solver<br>0=Defect correction (DC) method is chosen<br>1=Default GMRES method is chosen<br>Preconditioner<br>1 = `solver` + LU (order 2 x `alphaprecond`)<br>2 = `solver` + LU (ghostpoints eliminated)<br>3 = `solver` + MG-RB-`cyclet`(`nu(1)`,`nu(2)`)<br>The multigrid (MG) solver requires `GhostGridZ=1` <br>0 = Allan's<br> 1 = Ole's <br>GMRESmaxiterations<br>reltol<br>abstol<br>maxit<br>cyclet<br>nu(1)<br>nu(2)<br>MGmaxgrids                                                                                                                                                                                                                                                                                                                                               |
| 8                  | . <br>SFsol%HH<br>SFsol%h<br>SFsol%L<br>SFsol%T<br>SFsol%i_wavel_or_per<br>SFsol%e_or_s_vel<br>SFsol%i_euler_or_stokes<br>SFsol%n_h_steps<br>SFsol%n_four_modes                                                                                            | Stream function solution parameters<br>SFsol%HH<br>SFsol%h<br>Wavelength<br>SFsol%T<br>SFsol%i_wavel_or_per<br>SFsol%e_or_s_vel<br>SFsol%i_euler_or_stokes<br>SFsol%n_h_steps<br>SFsol%n_four_modes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| 9                  | .<br>StoreDataONOF<br>.<br>.<br>iKinematics<br>.<br>.<br>.<br>formattype<br>.<br>nOutFiles                                                                                                                                                                 | Data Storage<br>0 = No output<br> +N = Binary storage at stride N steps <br> -N = Ascii storage at stride N steps<br>Export kinematics <br>0 = don't output (default)<br>20 = Output kinematics to `Kinematics_**.bin`<br>30 = Output wave gauge kinematics to `waveGauges.dat` <br>0 = Binary <br> 1 = Unformatted <br>Number of output files                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| 10:10+`nOutFiles`  | .<br>Output(i)%xbeg<br>Output(i)%xend<br>Output(i)%xstride<br>Output(i)%ybeg<br>Output(i)%yend<br>Output(i)%ystride<br>Output(i)%tbeg<br>Output(i)%tend<br>Output(i)%tstride                                                                               | For each outfile                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 10                 | LinearONOFF<br>.<br>.<br>PressureTermONOFF<br>.<br>.<br>.<br>.                                                                                                                                                                                             | Applied free-surface pressure<br> 0=Linear model is employed <br> $\neq$ 0 = Fully nonlinear model is employed<br> Free surface pressure term <br>0= not applied <br>1=2D Gaussian surface pressure (`funPressureTerm.f90`)<br>2=3D Gaussian surface pressure ( `funPressureTerm.f90`)<br>2=3D tanh surface pressure ( `funPressureTerm.f90`)                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| 11                 | .<br>filteringONOFF<br>filterALPHA<br>filterORDER<br>sigma_filt(1)<br>sigma_filt(2)<br>sigma_filt(3)                                                                                                                                                       | SG-Filtering<br>>0 = Filtering applied every `filteringONOFF` time steps <br>SG(filterNP=2x`filterALPHA`+1, `filterORDER`)<br>SG(filterNP=2x`filterALPHA`+1, `filterORDER`)<br>sigma_filt(1)<br>sigma_filt(2)<br>sigma_filt(3)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| 12                 | .<br>BreakMod%i_breaking<br>.<br>BreakMod%T_half<br>BreakMod%tan_phi_b<br>BreakMod%tan_phi_0<br>BreakMod%del_fac<br>BreakMod%cel_fac<br>BreakMod%gamma_break                                                                                               | Breaking Model (opt)<br>$\neq$ 0 = Turn on <br>2 = Compute but don't apply <br>BreakMod%T_half<br>BreakMod%tan_phi_b<br>BreakMod%tan_phi_0<br>BreakMod%del_fac<br>BreakMod%cel_fac<br>BreakMod%gamma_break                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| 13                 | .<br>relaxONOFF<br>relaxTransientTime<br>relaxNo<br>relaxXorY<br>relaxDegrees                                                                                                                                                                              | Relaxation Zones<br>1 = Turn on<br>relaxTransientTime<br>Number of relaxation zones<br>relaxXorY<br>relaxDegrees                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 14:14+`relaxNo`    | . <br>RelaxZones(i)%BBox(1)<br>RelaxZones(i)%BBox(2)<br>RelaxZones(i)%BBox(3)<br>RelaxZones(i)%BBox(4)<br>RelaxZones(i)%ftype<br>RelaxZones(i)%param<br>RelaxZones(i)%XorY<br>RelaxZones(i)%WavegenOnOff<br>RelaxZones(i)%XorYgen<br>RelaxZones(i)%degrees | For each relaxation zone<br>x1 (zone bound)<br>x2 (zone bound)<br>y1 (zone bound)<br>y2 (zone bound)<br>= +/- 9,10; sign gives direction<br>param<br>XorY<br>WavegenOnOff<br>XorYgen<br>degrees                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| 15                 | .<br>PDampingONOFF<br>NDampZones                                                                                                                                                                                                                           | Pressure damping zones (opt) <br>$\neq$ 0 = On<br>Number of damping zones (only `NDampZones=1` supported)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| 16:16+`NDampZones` | .<br>PDampZones(i)%BBox(1)<br>PDampZones(i)%BBox(2)<br>PDampZones(i)%BBox(3)<br>PDampZones(i)%BBox(4)<br>PDampZones(i)%g0Phi<br>PDampZones(i)%g0Eta<br>PDampZones(i)%type<br>.                                                                             | For each damping zone<br>x1 (zone bound)<br>x2 (zone bound)<br>y1 (zone bound)<br>y2 (zone bound)<br>$\gamma_0$ (dynamic FSBC)<br>$\gamma_0$ (kinematic FSBC)<br>0 = friction on the velocity <br> 1 = friction on the potential                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 17                 | .<br>swenseONOFF<br>swenseTransientTime<br>swenseDir<br>West_refl<br>East_refl<br>North_refl<br>South_refl                                                                                                                                                 | SWENSE line<br>swenseONOFF<br>swenseTransientTime<br>swenseDir<br>West_refl<br>East_refl<br>North_refl<br>South_refl                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| 18                 | . <br>curvilinearONOFF<br>.                                                                                                                                                                                                                                | Curvilinear Input<br>0=Standard Cartesian model employed <br>1=Curvilinear model employed (3D cases only)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| 19                 | If `IncWaveType`=3<br> wave3DFlux%rampTime<br>wave3DFlux%order<br>wave3DFlux%inc_wave_file                                                                                                                                                                 | Wave generation with flux condition on western boundary<br> wave3DFlux%rampTime<br>wave3DFlux%order<br>wave3DFlux%inc_wave_file                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| 19                 | If `IncWaveType`=2<br>ispec<br>.<br>.<br>.                                                                                                                                                                                                                 | Irregular Wave Specification <br>0= PM<br>1=Normal JONSWAP with $\gamma$ = 3.3<br>2 = Based on input files<br>3 = JONSWAP with variable gamma value                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| 19                 | If `ispec`= 0, 1<br>ispec<br>Tp<br>Hs<br>h0<br>kh_max<br>seed<br>seed2<br>x0<br>y0                                                                                                                                                                         | Normal PM or JONSWAP spectrum <br>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| 19                 | If `ispec`= 2<br>ispec<br>Tp<br>Hs<br>h0<br>kh_max<br>seed<br>seed2<br>x0<br>y0<br>inc_wave_file                                                                                                                                                           | 2D Irregular waves with input file <br>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| 19                 | If `ispec`= 3<br>ispec<br>Tp<br>Hs<br>h0<br>kh_max<br>seed<br>seed2<br>x0<br>y0<br>gamma_jonswap                                                                                                                                                           | 2D irregular waves with non-standard gamma value                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 19                 | If `ispec`$\ge$ 30<br>ispec<br>Tp<br>Hs<br>h0<br>kh_max<br>seed<br>seed2<br>x0<br>y0<br>inc_wave_file<br>beta0<br>s0<br>gamma_jonswap                                                                                                                      | multi-directional irregular waves<br>ispec<br>Tp<br>Hs<br>h0<br>kh_max<br>seed<br>seed2<br>x0<br>y0<br>inc_wave_file<br>Heading angle<br>Spreading factor<br> JONSWAP $\gamma$                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |

<br>
<br>
<br>

## 6. OceanWave3D.init file

At the end of a simulation the code produces a restart file called `OceanWave3D.end`. To restart from this end position, the name of this file should be changed to `OceanWave3D.init` and set `IC=-1` in the `OceanWave3D.inp` file. The file format is given below.

| Line number                | Variable                                            | Description                                                                                                                                                                                                                                                                                         |
| -------------------------- | --------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1                          | HEAD(3)                                             | Header line                                                                                                                                                                                                                                                                                         |
| 2                          | xtankIC<br>ytankIC<br>nxIC<br>nyIC<br>t0IC          | $\equiv$ Lx in OceanWave3D.inp <br>$\equiv$ Ly in OceanWave3D.inp <br> $\equiv$ FineGrid%Nx in OceanWave3D.inp (if not equivalent will complain on restart)<br> $\equiv$ FineGrid%Ny in OceanWave3D.inp (if not equivalent will complain on restart) <br> $\equiv$ time0 (initial time for restart) |
| 3 : 3+`nxIC`$\times$`nyIC` | .<br>WaveField%E(i,j)<br>WaveField%P(i,j)<br>.<br>. | For i each free-surface nodal location, read the <br> Wave Elevation <br> Potential (Scalar) Field <br> The start index for i is 1+`GhostGridY` <br> The start index for j is 1+`GhostGridX`                                                                                                        |

<br>
<br>
<br>

## 6. Bathymetry file (fname_bottom)

This file specifies the topology of the seabed as a function of nodal position. The format of the file is given below.

| Line number                              | Variable                                                                                                    | Description                                                                                                                                                                         |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| 1                                        | HEAD(4)                                                                                                     | Header line                                                                                                                                                                         |
| 2                                        | DetermineBottomGradients                                                                                    | 0 = Read gradients from file <br> 1 = Determine gradients from depth profile on file                                                                                                |
| 3 : 3+`FineGrid%Nx`$\times$`FineGrid%Ny` | FineGrid%h(i,j)<br>FineGrid%hx(i,j)<br>FineGrid%hxx(i,j)<br>FineGrid%hy(i,j)<br>FineGrid%hyy(i,j)<br>.<br>. | Depth, $h$<br> $\partial_x h$<br>$\partial_{xx} h$<br>$\partial_y h$<br>$\partial_{yy} h$ <br> The start index for i is 1+`GhostGridY` <br> The start index for j is 1+`GhostGridX` |     |

# Output File Formats

## Random wave coefficients file (eta0_coeffs)

This file contains the Fourier coefficients using the chosen Spectrum, where it is hard-coded that the Fourier amplitudes lie exactly on the the spectrum, but there is commented-out code to make them Gaussianly distribute around the spectrum (see SUBROUTINES `random_wave_coefficients` and `build_coeffs`).

For a unidirectional wave (see e.g. [Faltinsen])

$$
\eta(t) = \sum_{j=1}^N A_j \cos(\omega_jt - k_jx + \psi_j)
$$

where

$$
\frac{1}{2}A_j^2 = S(\omega_j)\delta\omega
$$

$$
\rightarrow \eta(t) = \sum_{j=1}^N [2 S(\omega_j)\delta\omega]^{0.5} \cos(\omega_jt - k_jx + \psi_j)
$$

Which at the influx boundary ($x=0$) is equivalent to:

$$
\eta(t) = \Re \left(\sum_{j=1}^N \Eta_j(\omega_j) e^{i\omega_jt}\right)
$$

where the complex coefficients $H_j$ are

$$
\Eta_j(\omega_j) = [2 S(\omega_j)\delta\omega]^{0.5} e^{i\psi_j}
$$

<br>

The exported file format is given below.

| Line number | Variable                                                            | Description                                                                        |
| ----------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| 1           | HEAD(4)                                                             | Header line                                                                        |
| 2:2+$N/2$   | $\delta\omega/(2\pi)$<br>$\mid H_j\mid$<br>$\Re(H_j)$<br>$\Im(H_j)$ | Frequency [Hz] <br> Amplitude of wave component <br> Real part <br> Imaginary part |

<br>
<br>

## Wave elevation file at inlet (eta0_irregular)

This file contains $t$ and $\eta_(\vec{x}=0,t)$ where

$$
\eta(t) = \Re \left(\sum_{j=1}^N \Eta_j(\omega_j) e^{i\omega_jt}\right) = \mathscr{F}(\Eta_j(\omega_j))
$$

In the code, the main FFT routine is `DREALFT` which can operate as a FFT and IFFT depending on the third flag.

## Fort.N files

For the 2D regular case in the example files, these are of length `Nx+2` where the +2 likely represents some ghost cell at the 2 ends of the domain. The number of output files is given by `nSteps`$\times$`dt` / `StoreDataONOFF` in `OceanWave3D.inp`.
