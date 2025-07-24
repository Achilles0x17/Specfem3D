# Specfem3D
run:
```
git clone --recursive --branch devel https://github.com/SPECFEM/specfem3d_globe.git
```
sbatch script template of f1:
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

# your code ...
```
First configure the source code: 
``` sh
module load gcc/11.2.0 openmpi/4.1.6
cd /home/achilles0x17/specfem3d_globe
./configure FC=gfortran CC=gcc MPIFC=mpif90
```
Then modify the Par_file   
Requirements:
- NPROC_XI = NPROC_ETA
- 6 * NPROC_XI * NPROC_ETA = total_rank
- NEX must be multiple of 8 * NPROC
- Add : OUTPUT_SEISMOS_HDF5	    = .false.

Next compile and run the mesher:

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
cd /home/&USERNAME/specfem3d_globe
# ./configure FC=gfortran CC=gcc MPIFC=mpif90
make meshfem3D
mpirun -np 24 ./bin/xmeshfem3D
```
check that you have ./OUTPUT_FILES/values_from_mesher.h
