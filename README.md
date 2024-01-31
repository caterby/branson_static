# branson_static
This repository contains scripts to set up and execute the static branson.

## 1. SETUP
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
 ```
## 2. Install the static OpenMPI
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


## 3. Install the static MPI

```
cd /workspace
wget https://www.mpich.org/static/downloads/4.1.2/mpich-4.1.2.tar.gz
tar xzvf mpich-4.1.2.tar.gz
cd mpich-4.1.2

# static mpicc 
./configure --prefix=/workspace/mpich-4.1.2-install --enable-static
make && sudo make install 
export MPI_HOME=/workspace/mpich-install
export PATH=$MPI_HOME/bin:$PATH
export LD_LIBRARY_PATH=$MPI_HOME/lib:$LD_LIBRARY_PATH
```


 **Example:**
 To generate 100GB of data, a set of queries (22 in total), load the data into the database, warm up the database, and run both the power and throughput tests, use:
 ```
 sudo ./run-tpch.sh -d 100 -q 1 -l -w -p -t
 ```
 **NOTE:** The resulting database, post data loading, will be approximately 3 times larger than the generated data. Ensure adequate storage space is available.

 Executing run-tpch.sh will automatically start the Prometheus SQL Exporter, which exposes metrics from DBMSs for Prometheus monitoring on port 9399. Remember to add this exporter to your Prometheus configuration and adjust the hostname to match your setup:

 **NOTE:** By default, Docker is configured to use the standard gateway for the Docker bridge network, associated with the IP address 172.17.0.1. Should there be any alterations to this setup, the sql_exporter configuration file will require modifications.
 ```
  # sql exporter metrics exporter scrape
  - job_name: “sql-exporter-server"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["machine-name.domain.com:9399"]
        labels:
          group: "cxl"
 ```
 For more information about this exporter, check the following link:
 [Prometheus SQL Exporter](https://github.com/free/sql_exporter)

 Additionally, the script provides an option to initiate the cAdvisor exporter, granting insights into container resource utilization and performance. This daemon aggregates and exports data about active containers on port 8082. Add this exporter to your Prometheus configuration and adjust the hostname to match your setup:
 
 ```
  # cAdvisor metrics exporter scrape
  - job_name: “cadvisor-server"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["machine-name.domain.com:8082"]
        labels:
          group: "cxl"
 ```
 For more information about this exporter, check the following link:
 [cAdvisor (Container Advisor)](https://github.com/google/cadvisor)
 
