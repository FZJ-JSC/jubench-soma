# Description

SOMA offers high-performance Monte-Carlo simulations of the "Single Chain in
Mean Field" model for soft coarse-grained polymers. The physical model for the
simulation is described [in a publication](https://doi.org/10.1063/1.2364506). The source
code for the software can be found
[on Gitlab](https://gitlab.com/InnocentBug/SOMA), while the description of the code
itself is described
[in an associated paper](https://doi.org/10.1016/j.cpc.2018.08.011).

The code is written in C99 and is parallelized with OpenACC, OpenMP, and MPI.
The performance of the benchmark is measured in terms time steps of simulation
per second (TPS). TPS is expected to scale linearly with the number of GPUs.

## Source

Archive Name : `soma-bench.tar.gz`

The file holds instructions to run the benchmark, according JUBE scripts, and configuration files to
run SOMA.

Sources for the SOMA application (tag 0.7.0) are to be cloned from the Git repository:

- repo : https://gitlab.com/InnocentBug/SOMA/
- tag: 0.7.0;
- SHA1SUM: 227ee4e86202ab58dba12e7592d0fc58a5efefef.

The clone is to be located to `src/soma/`. See the following snippet to clone SOMA:

```
git clone --branch 0.7.0 https://gitlab.com/InnocentBug/SOMA.git src/soma
```

### JUBE

The JUBE step `fetch` clones the source to SOMA.

## Building

The benchmark for SOMA is intended for the GPU version only. SOMA needs the following software to build:

1. An OpenACC-Supporting C Compiler
2. A CUDA Toolkit
3. Doxygen
4. An MPI Installation
5. MPI-Enabled Version of HDF5
6. CMake (version 3.20 or later)
7. Python
8. `h5py` Python Module

Once the dependencies are in place, SOMA can be built and installed with the following commands.


```
cmake -B build -S SOMA -DINSTALL_PYTHON=ON
cmake --build build
cmake --install build --prefix install
```

This will install the SOMA executables and the related Python scripts to the `install` directory.

Once SOMA is built and installed, the input data for the simulation can be generated from the
`src/coord.xml` file.  The following command will generate the required `src/coord.h5` file.

```
install/python-script/ConfGen.py -i src/coord.xml
```


### Modification

- Changing the workload is not within scope.
- Floating point optimisation must not be set to `--ffast-math` or similar.

### JUBE

The JUBE step `compile` takes care of building the benchmark. It configures, builds, and installs the
benchmark.

## Execution

Changing the workload is not within scope.

### Command Line

On the command line the benchmark can be called with

```
[mpiexec] ./SOMA --coord-file src/coord.h5 \
  --timesteps 20000 --gpus 4 --final-file /dev/null --rng-seed 2167612478
```

### JUBE

To submit a self-contained benchmark run to the batch system, call `jube run
benchmark/jube/default.xml`. JUBE will generate the necessary configuration and files, and submit
the benchmark to the batch engine.

The following parametes of the JUBE script might need to be adapted:

- `taskspernode`: must be equal to the number of GPUs on a node
- `queue`: SLURM queue to use

## Verification

The application reports `SOMA finished execution without errors.` to indicate successful execution without errors.

## Results

The benchmark prints an `effective_runtime` runtime and a TPS (_Timesteps Per Second_) for the run
step. The metric `effective_runtime` is used as a measure of performance of the benchmark (lower is
better).

### JUBE

Using `jube analyse` and a subsequent `jube result` prints an overview table with the number of
nodes, tasks per node, `effective_runtime`,  and the TPS. The runtime metric is shown as
`effective_runtime`.


## Baseline

The baseline configuration must be chosen such that the effective runtime is less than or equal to
54 s. This value is achieved with 8 nodes on the JUWELS Booster system at JSC.

nodes | taskspernode |  TPS | effective runtime (s) |
----- | ------------ | ---         | --- |
8     | 4            |  389.993    | 52 |
