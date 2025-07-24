# Specfem3D 

First run:
```
git clone --recursive --branch devel https://github.com/SPECFEM/specfem3d_globe.git
```
## CPU
Then modify the Par_file
Requirements:
- NPROC_XI = NPROC_ETA
- 6 * NPROC_XI * NPROC_ETA = total_rank
- NEX must be multiple of 8 * NPROC
- Add : OUTPUT_SEISMOS_HDF5	    = .false.
- RECORD_LENGTH_IN_MINUTES = 4d0

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
## GPU
```sh
#! /bin/bash
#SBATCH -e error_%j.log
#SBATCH -o output_%j.log
# ===== 此處 Slurm 參數請勿更動 =====
#SBATCH --job-name=specfem3D
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --time=00:06:00
#SBATCH --account="ACD114003"
# ===================================

# ===== 調整 GPU (MPI Rank) 數量 =====
#SBATCH --gres=gpu:6
#SBATCH --ntasks-per-node=6
# ====================================


module load nvhpc-24.11_hpcx-2.20_cuda-12.6 gcc10/10.2.1

cd /home/achilles0x17/specfem3d_globe

git submodule update --init --recursive

make clean

./configure --with-cuda=cuda9 CUDA_LIB="$NVHPC_ROOT/cuda/lib64" \
CUDA_INC="$NVHPC_ROOT/cuda/include" --enable-cuda-aware-mpi

make meshfem3D
mpirun -np 6 ./bin/xmeshfem3D 

make specfem3D

ml miniconda3/conda24.5.0_py3.9 gcc10/10.2.1 nvhpc-24.11_hpcx-2.20_cuda-12.6
ml cuda/12.8
# To do profiling, uncomment below
# PROFILER="nsys profile --trace cuda,nvtx,openmp,mpi --delay 10 --duration 10 --output specfem3D-profile-${SLURM_JOB_ID}.nsys-rep"
# ml cuda/12.8

$PROFILER \
    mpirun -np 6 ./bin/xspecfem3D
```
Par_file:
- USE_GPU = .true.


check that you have  
- ./OUTPUT_FILES/values_from_mesher.h
- ./OUTPUT_FILES/output_mesher.txt
- ./OUTPUT_FILES/output_solver.txt

Create movie:
```
module load gcc/11.2.0 openmpi/4.1.6
make create_movie_AVS_DX
./bin/xcreate_movie_AVS_DX
```
Input: 
- choose 5:  create files in VTU format with individual files  
- First timestamp: 1  
- Last timestamp: -1  
- choose one (1=Z, 2=N, 3=E):  

Download the .vtu files in ./OUTPUT_FILES and open ParaView  
- File > Open
- 選擇 VTU 檔案
- 點擊左邊的 Apply
- Properties 中的 Coloring 選擇 val_[Z/N/E]
- 點擊工具欄的 Play



