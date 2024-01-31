# branson_static
This repository contains scripts to set up and execute the static branson.

## 1. Setup
Build the directory for the installation:
```
mkdir workspace && cd workspace
```
Setup the environment, install the following libraries and the necessary tools:
 ```
apt-get update
apt-get install git wget build-essential python3 python3-pip vim libpmix-dev meson gperf libcap-dev pkg-config libmount-dev
pip3 install jinja2
 ```

**Note:** Make sure the version of your C/C++ compiler with suppport for C11 and C++14, and the version of CMAKE is 3.9+. To check the support for C11 and C++14, you can use the following command:
```
gcc --version
g++ --version
```
Install autoconf tool for the compilation (autoconf ≥ 2.71):
```
autoconf --version
wget https://ftp.gnu.org/gnu/autoconf/autoconf-2.71.tar.xz
tar -xf autoconf-2.71.tar.xz
cd autoconf-2.71/
./configure
time make 
sudo make install 
. ~/.profile
autoconf --version    
```

## 2. Install the static OpenMPI
-- The MPI is REQUIRED:
 * MPI, A High Performance Message Passing Library, <http://www.open-mpi.org/>
   A parallel communication library is required in BRANSON.
```
cd /workspace
wget https://download.open-mpi.org/release/open-mpi/v5.0/openmpi-5.0.1.tar.gz
gunzip -c openmpi-5.0.1.tar.gz | tar xf -
mkdir openmpi-5.0.1-install

# Disable unneeded features to reduce the install time
cd openmpi-5.0.1/
./configure --prefix=/workspace/openmpi-5.0.1-install --without-memory-manager --disable-dlopen --enable-static --disable-shared --with-psm2=no --with-psm=no --with-ofi=no --without-verbs --without-rdmacm --without-libnuma
make -j32 all
make -j32 install

# Change the environment directory for static OpenMPI
export MPI_HOME=/workspace/openmpi-5.0.1-install/bin
export PATH=$MPI_HOME/bin:$PATH
export LD_LIBRARY_PATH=$MPI_HOME/lib:$LD_LIBRARY_PATH
```
## 3. Install other static libs
In our machine, we installed libudev as an example, if other libs also needed in your machine, please follow the same way to install them.
```
cd /workspace/
git clone https://github.com/systemd/systemd-stable.git
cd systemd-stable
./configure --auto-features=disabled --default-library=static -D standalone-binaries=true -D static-libsystemd=true -D static-libudev=true -D link-udev-shared=false -D link-systemctl-shared=false -D link-networkd-shared=false -D link-timesyncd-shared=false
make
find ./ -name "libudev*"
cp ./build/libudev.a /usr/local/lib/
```

## 4. Install other dependencies (Optional)
```
-- The following packages are OPTIONAL:

 * caliper, CALIPER, <https://software.llnl.gov/Caliper/>
   Code instrumentation for performance analysis
 * METIS, METIS, <http://glaros.dtc.umn.edu/gkhome/metis/metis/overview>
   METIS is a set of serial programs for partitioning graphs,
   partitioning finite element meshes, and producing fill reducing orderings for
   sparse matrices.
 * HDF5, HDF5 is a data model, library, and file format for storing
   and managing data. It supports an unlimited variety of datatypes, and is
   designed for flexible and efficient I/O and for high volume and complex
   data., <https://support.hdfgroup.org/HDF5/>
   Provides optional visualization support for Branson.
```
**NOTE:** By default, we didn't compile these dependencies into static. To skip these dependencies, just set the related cmake options when building Branson.

## 5. Install branson
```
# download branson
sudo apt update
sudo apt install cmake
cd /workspace
git clone https://github.com/lanl/branson.git
cd /workspace/branson/src/
# disable optional dependenies (Metis, HDF5, etc)
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/workspace/branson/src -DCMAKE_PREFIX_PATH=/workspace/openmpi-5.0.1-install/bin/ -DUSE_OPENMP=ON 
make -j

# After the compiliation, check if the BRANSON is static.
cd /workspace/branson/
ldd BRANSON
# If it shows `not a dynamic executable`, it is static.
# Run BRANSON, usage: BRANSON <path_to_input_file>
./BRANSON 

# If compilation failed or the executable binary is dynamic, use the instructions below to resolve isssues
1) make -n
# The order of libraries is crucial for static linking.
mpic++ -static BRANSON.o -o BRANSON -lopen-pal -lpmix -lhwloc -ludev -lz -ldl -lltdl -lrt -lc  -lgcc /workspace/AMG2023-self-contained/hypre-2.30.0/src/hypre/lib/libHYPRE.a -luuid -levent -lutil -lnuma -lm -lrt --showme
# reorganize the order of library
g++ -static BRANSON.o -o BRANSON -lopen-pal -lpmix -lhwloc -ludev -lz -ldl -lltdl -lrt -lc -lgcc -luuid -levent -lutil -lnuma -lm -lrt -I/workspace/openmpi-5.0.1-install/include -L/workspace/openmpi-5.0.1-install/lib -Wl,-rpath -Wl,/workspace/openmpi-5.0.1-install/lib -Wl,--enable-new-dtags -lmpi -lopen-pal -lpmix -lz -lm -levent_core -levent_pthreads -lhwloc -lz -levent_core -levent_pthreads -lhwloc -ludev -lrt
```
