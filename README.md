# Specfem3D
First:
```
git clone --recursive --branch devel https://github.com/SPECFEM/specfem3d_globe.git
```
Then modify the Par_file
Requirements:
- NPROC_XI = NPROC_ETA
- 6 * NPROC_XI * NPROC_ETA = total_rank
- NEX must be multiple of 8 * NPROC
- Add : OUTPUT_SEISMOS_HDF5	    = .false.

Next execute the script using sbatch on f1:
```sh
#!/bin/bash
#SBATCH --job-name=specfem3d
#SBATCH --nodes=1
#SBATCH --time=00:30:00
#SBATCH --partition=ct112
#SBATCH --account=ACD114003
#SBATCH -n 112
#SBATCH -e error_%j.log
#SBATCH -o output_%j.log

# max number of cpus per node is 112

module load gcc/11.2.0 openmpi/4.1.6

cd /home/achilles0x17/specfem3d_globe

git submodule update --init --recursive

make clean

# First configure the source code
./configure FC=gfortran CC=gcc MPIFC=mpif90

# Next compile and run the mesher
make meshfem3D
mpirun -np 24 ./bin/xmeshfem3D

# Compile and run the solver
make specfem3D
mpirun -np 24 ./bin/xspecfem3D
```

check that you have  
- ./OUTPUT_FILES/values_from_mesher.h
- ./OUTPUT_FILES/output_mesher.txt
- ./OUTPUT_FILES/output_solver.txt
