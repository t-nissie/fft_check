fft_check.F, a 3-dimensional FFT benchmark program written in Fortran
=====================================================================
fft_check.F is a benchmark program. It times
in-place double-precision complex 3-dimensional FFT and
in-place and out-of-place double-precision real 3-dimensional FFT.
It is written in Fortran and parallelized with OpenMP.

Homepage and download
---------------------
http://loto.sourceforge.net/feram/src/fft_check.html is the homepage of fft_check.

Its GPLed source code is in feram-X.YY.ZZ/src/ of the feram package.
You can freely download a tar ball of feram (feram-X.YY.ZZ.tar.xz) from
http://sourceforge.net/projects/loto/files/feram/ .
feram is molecular dynamics (MD) simulator for bulk and
thin-film ferroelectrics and GPLed free software.

Home page of feram is http://loto.sourceforge.net/feram/ .

== How to build fft_check
Although fft_check can be built by the usual 'configure && make fft_check' manner
along with feram (see ../INSTALL), makefiles for several compilers and
architectures are provided in names of fft_check.Makefile.*.
You can build fft_check with GNU Fortran (gfortran), Intel Fortran (ifort),
IBM XL Fortran (xlf90_r), etc. You can link FFTW, Intel MKL, Hitachi MATRIX/MPP
as an FFT library. For example,
 $ make -f fft_check.Makefile.Intel-gfortran-fftw3_omp

=== FFTW and wisdom file
The version number of your FFTW library can be obtained with
fftw-wisdom command as
 $ fftw-wisdom -V

If fft_check.F is compiled with FFTW library and f90_wisdom.f,
it imports 'wisdom' file in current directory or /etc/fftw/wisdom
and exports 'wisdom_new' into current directory.

=== AMD Core Math Library (ACML)
Use fft_acml.f and fft_acml.Makefile in feram-0.22.05.tar.xz.
 $ make -f fft_acml.Makefile
Note that FFT in ACML is currently not so good:
 * Slow.
 * Inaccurate. Try it with TOLERANCE = 1.0d-12.
 * zdfft3d() normalizes transformed matrix. It is not a standard behavior.

== How to execute fft_check
Without OMP_NUM_THREADS environment variable,
 $ ./fft_check 10000 80 90 100
fft_check normally uses all cores in your computer,
where it FFT an array of size 80x90x100 in 10,000 iterations.
Note that the number of iterations does not affect wisdom_new file.

If you want to benchmark efficiency of single processor on a
multi-processor system, use taskset(1) on Linux, cpuset(1)
on FreeBSD, dplace(1) on SGI or pbind(1) on Solaris for
binding threads to cores on one processor. For example, on a
system with two hyper-threading-off Xeon X5650,
 $ OMP_NUM_THREADS=6 taskset -c 0-5 ./fft_check 10000 80 90 100

numactl(8) on Linux is also a useful command for achieving
good performance on Non-Uniform Memory Access (NUMA) systems.
 $ numactl --help
 $ numactl --show
 $ numactl --hardware
 $ numactl --cpunodebind=0,1 --interleave=all ./fft_check 100 256 256 256

== Output, timing and GFLOPS
Verbose reports will be written into standard error (STDERR).
Formatted results will be written into standard output (STDOUT)
in an order of
N_TIMES Lx Ly Lz N NTHREADS
plan_ci time_ci GFLOPS_ci
plan_ri time_ri GFLOPS_ri
plan_ro time_ro GFLOPS_ro,
where plan denotes time in second for planning,
time denotes time in second for FFT,
_ci denotes in-place double-precision complex 3-dimensional FFT,
_ri denotes in-place double-precision real 3-dimensional FFT, and
_ro denotes out-of-place of that.

Giga FLOPS values are roughly estimated from
5*N*log_2(N) floating point operations.

== Single node results and conditions
In Fig:powr2 and Fig:nonp2,
results of 3-dimensional FFT benchmark
on single node of some systems are shown.
Computational conditions are listed below.

\Fig:powr2 19example-fft-benchmark/fft_powr2.jpg
Results of 3-dimensional FFT benchmark of powers of two.
(a) Double-precision complex, in-place.
(b) Double-precision real, in-place.
(c) Double-precision real, out-of-place.
Intel MKL is used in "X7560/MKL" benchmark.
FFTW is used in others.
/Fig:powr2

\Fig:nonp2 19example-fft-benchmark/fft_nonp2.jpg
Results of 3-dimensional FFT benchmark of non-powers of two.
(a) Double-precision complex, in-place.
(b) Double-precision real, in-place.
(c) Double-precision real, out-of-place.
Intel MKL is used in "X7560/MKL" benchmark.
FFTW is used in others.
/Fig:nonp2

Raw data of results of benchmark are
19example-fft-benchmark/fft_check_powr2.*.dat and
19example-fft-benchmark/fft_check_nonp2.*.dat.
They are plotted with GNUPLOT scripts of
19example-fft-benchmark/fft_powr2.gp and
19example-fft-benchmark/fft_nonp2.gp.

=== Hitachi SR16000 model M1
 * 1 node = 4 chips of 3.83 GHz POWER7
 * 32 core = 8 core * 4 chip (NUMA)
 * Memory: DDR3 1333 MHz
 * SMT off (SMT: Simultaneous Multithreading)
 * Makefile: fft_check.Makefile.SR16000-xlf90_r-fftw_xlc
 * FFTW planner flag: FFTW_PATIENT
 * Environment variables: MALLOCMULTIHEAP=true, LDR_CNTRL="LARGE_PAGE_DATA=M",
   OMP_NUM_THREADS=32, XLSMPOPTS="spins=0:yields=0:parthds=32:stride=2:startproc=0"
 * FFTW 3.3.2 compiled with xlc: ../configure CC=xlc_r F77=xlf_r
   CFLAGS='-O3 -qansialias -w -qarch=auto -qtune=auto -qsmp=omp'
   OPENMP_CFLAGS=-qsmp=omp --host=power --enable-openmp --enable-threads
   --enable-fma --prefix=/sysap/fftw_xlc/fftw-3.3.2a
 * Hitachi OFORT90 and FFT subroutines in MATRIX/MPP
   are slower than xlc, xlf90_r and FFTW.

=== Intel Xeon X7560
 * 1 node = 4 chips of 2.27 GHz X7560
 * 32 core = 8 core * 4 chip (NUMA)
 * Memory: DDR3 1066 MHz
 * Mother board: NEC Express5800
 * HT off (HT: Hyper Threading)
 * Makefile: fft_check.Makefile.Intel-ifort-MKL or fft_check.Makefile.Intel-ifort-fftw3_omp
 * Environment variables: OMP_NUM_THREADS=32
 * How to execute: ../fft_check 50 400 400 400
 * Note that, in the case of real-FFT with MKL,
   out-of-place is much slower than in-place.
 * There is a numerical error problem in MKL.
   Check that with TOLERANCE=1.0d-15.

=== Intel Xeon E5-2680
 * 1 node = 2 chips of 2.70 GHz E5-2680
 * 16 core = 8 core * 2 chip (NUMA)
 * Memory: DDR3 1333 MHz
 * Mother board: Supermicro X9DRW
 * HT off (HT: Hyper Threading)
 * Makefile: fft_check.Makefile.Intel-gfortran-fftw3_omp
 * FFTW planner flag: FFTW_PATIENT
 * Environment variables: OMP_NUM_THREADS=16
 * How to execute: numactl --cpunodebind=0,1 --interleave=all ../fft_check 50 400 400 400
 * FFTW 3.3.2 compiled with gfortran 4.4: ../configure --prefix=/usr/local
   --libdir=/usr/local/lib64 --enable-openmp --enable-threads --enable-sse2
   --enable-avx --enable-shared

=== Intel Xeon X5650
 * 1 node = 2 chips of 2.67 GHz X5650
 * 12 core = 6 core * 2 chip (NUMA)
 * Memory: DDR3 1333 MHz
 * Mother board: Supermicro X8DTG-D
 * HT off (HT: Hyper Threading)
 * Makefile: fft_check.Makefile.Intel-gfortran-fftw3_omp
 * FFTW planner flag: FFTW_PATIENT
 * Environment variables: OMP_NUM_THREADS=12
 * How to execute: ../fft_check 50 400 400 400
 * FFTW 3.3.2 compiled with gfortran 4.4: ../configure --prefix=/usr/local
   --libdir=/usr/local/lib64 --enable-openmp --enable-threads --enable-sse2
   --enable-shared

=== Fujitsu FX10
 * 1 node = 1 chip of 1.85 GHz SPARC64 IXfx
 * 16 core
 * Memory: DDR3 1333 MHz
 * Makefile: fft_check.Makefile.FX10-frtpx-fftw3_omp
 * FFTW planner flag: FFTW_MEASURE
 * Environment variables: FLIB_FASTOMP=TRUE
 * How to execute: ../fft_check 50 400 400 400
 * FFTW 3.3.x
 * CAUTION: FFT in Fujitsu SSLII is much slower than FFTW.

== Single chip results and conditions
In Fig:powr2chip and Fig:nonp2chip,
results of 3-dimensional FFT benchmark
on single chip 3.83 GHz POWER7 and 2.70 GHz E5-2680.
For small size FFT, single chip may give better efficiency than single node.
Computational conditions are listed below.

Benchmarks with Tesla K20X and Tesla M2090 GPUs are also plotted in
In Fig:powr2chip (b) and Fig:nonp2chip (b).
Double-precision real竊把omplex 3-dimensional in-place FFT is performed
on the GPU devices with cufft_check.F and cufft_module.f.


\Fig:powr2chip 19example-fft-benchmark/fft_powr2chip.jpg
Results of 3-dimensional FFT benchmark of powers of two with single chip.
(a) Double-precision complex, in-place.
(b) Double-precision real, in-place.
(c) Double-precision real, out-of-place.
FFTW is used.
/Fig:powr2chip

\Fig:nonp2chip 19example-fft-benchmark/fft_nonp2chip.jpg
Results of 3-dimensional FFT benchmark of non-powers of two with single chip.
(a) Double-precision complex, in-place.
(b) Double-precision real, in-place.
(c) Double-precision real, out-of-place.
FFTW is used.
/Fig:nonp2chip

Raw data of results of benchmark are
19example-fft-benchmark/fft_check_powr2chip.*.dat and
19example-fft-benchmark/fft_check_nonp2chip.*.dat.
They are plotted with GNUPLOT scripts of
19example-fft-benchmark/fft_powr2chip.gp and
19example-fft-benchmark/fft_nonp2chip.gp.

=== Hitachi SR16000 model M1
 * Single chip of 3.83 GHz POWER7
 * Memory: DDR3 1333 MHz
 * SMT off (SMT: Simultaneous Multithreading)
 * FFTW planner flag: FFTW_PATIENT
 * SC queue in CCMS-IMR is used.
 * Environment variables:

=== Intel Xeon E5-2680
 * Single chip of 2.70 GHz E5-2680
 * Memory: DDR3 1333 MHz
 * Mother board: Supermicro X9DRW
 * HT off (HT: Hyper Threading)
 * Makefile: fft_check.Makefile.Intel-gfortran-fftw3_omp
 * FFTW planner flag: FFTW_PATIENT
 * How to execute: numactl --cpunodebind=0 ../fft_check 50 400 400 400

== Comment on "padding"
If the numbers of dimensions of an array are powers of two,
for example a(512,512,512), "bank conflict" may occur in FFT and
it reduces computational speed. To avoid "bank conflict",
"padding" is commonly introduced, for example a(512,512+3,512).
However, introduction of "padding" make code complicated.
Therefore, "padding" is not introduced in this fft_check.F.

See feram_fftw_wisdom.F for an example of usage of padding and
fftw_plan_many_*().

== fft_check_mpi.F, an MPI-parallelized large-scale FFT benchmark program
You can find an MPI-parallelized large-scale FFT benchmark program at
https://github.com/t-nissie/fft_check_mpi .
It is written in Fortran, using FFTW, parallelized with MPI.

== Copying and author
Copyright ﾂｩ 2007-2015 by Takeshi Nishimatsu

fft_check is distributed in the hope that
it will be useful, but WITHOUT ANY WARRANTY.
You can copy, modify and redistribute fft_check,
but only under the conditions described in
the GNU General Public License (the "GPL").

Takeshi Nishimatsu (t-nissie{at}imr.tohoku.ac.jp)
