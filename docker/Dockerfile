FROM gcc:latest

## Additional packages
RUN echo "deb http://deb.debian.org/debian bookworm-backports main" >> /etc/apt/sources.list.d/debian-12-backports.list
RUN apt-get upgrade -y
RUN apt-get update -y
RUN apt-get install gdb -y
RUN apt-get install cmake -y
RUN apt-get install valgrind -y
RUN apt-get install vim -y
RUN apt-get install neovim -y
RUN apt-get install git -y
RUN apt-get install libc++-19-dev -y
RUN apt-get install libhdf5-dev -y
RUN apt-get install python3-numpy -y
RUN apt-get install python3-scipy -y
RUN apt-get install python3-matplotlib -y
RUN apt-get install python3-h5py -y
RUN apt-get install libomp-19-dev -y
RUN apt-get install h5utils -y
RUN apt-get install hdf5-tools -y
RUN apt-get install hdf5-helpers -y
RUN apt-get install libgsl-dev -y
RUN apt-get install libcatch2-dev -y
RUN apt-get install parallel -y
RUN git clone https://github.com/dmtcp/dmtcp.git /opt/dmtcp && \
    cd /opt/dmtcp && \
    ./configure && \
    make -j$(nproc) && \
    make install

#### RUN apt-get install openmpi-bin libopenmpi-dev -y

RUN useradd -ms /bin/bash user # openmpi does not like to be executed as root

## Install MPI

## Install Eigen
RUN mkdir /opt/Eigen/
RUN cd /opt/Eigen && git clone https://gitlab.com/libeigen/eigen.git
RUN mkdir /opt/Eigen/build/
RUN cd /opt/Eigen/build/ && cmake ../eigen/ -DCMAKE_INSTALL_PREFIX="./install" -DINCLUDE_INSTALL_DIR="./install/include" && make install
RUN rm -rf /opt/Eigen/eigen/.git

## Install HighFive 
RUN mkdir /opt/HighFive/
RUN cd /opt/HighFive && git clone --recursive https://github.com/highfive-devs/highfive.git 
RUN cd /opt/HighFive
RUN cd /opt/HighFive/highfive && cmake cmake -DCMAKE_INSTALL_PREFIX=build/install -B build .
RUN cd /opt/HighFive/highfive && cmake --build build --parallel
RUN cd /opt/HighFive/highfive && cmake --install build
RUN rm -rf /opt/HighFive/highfive/.git

### Install system dependencies
#RUN apt-get update && apt-get install -y wget gnupg curl lsb-release #software-properties-common
#
### Add Intel GPG key securely
#RUN curl -fsSL https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB | gpg --dearmor -o /usr/share/keyrings/intel-oneapi.gpg
#
### Add Intel oneAPI APT repository
#RUN echo "deb [signed-by=/usr/share/keyrings/intel-oneapi.gpg] https://apt.repos.intel.com/oneapi all main" > /etc/apt/sources.list.d/intel-oneapi.list
#
### Install MKL (runtime + dev headers)
#RUN apt-get update && apt-cache search oneapi || true
#RUN apt-get update && apt-get install -y intel-oneapi-mkl intel-oneapi-mkl-devel



### SLEPC/PETSC
### options
ENV LIBS=/opt
ENV OPTFLAGS="-O3 -march=native -mtune=native"


### clone petsc
ENV PETSC_DIR=$libs/petsc
RUN mkdir -p $LIBS && cd $LIBS && git clone -b release https://gitlab.com/petsc/petsc.git petsc && cd $PETSC_DIR && git checkout v3.23.4


### clone slepc
ENV SLEPC_DIR=$LIBS/slepc
RUN cd $LIBS && git clone https://gitlab.com/slepc/slepc slepc && cd $SLEPC_DIR && git checkout v3.23.0



### complex version
ENV PETSC_ARCH=petsc_complex
RUN cd $PETSC_DIR


RUN cd $PETSC_DIR && ./configure --with-scalar-type=complex  --COPTFLAGS=$OPTFLAGS --CXXOPTFLAGS=$OPTFLAGS --FOPTFLAGS=$OPTFLAGS --with-64-bit-indices --with-debugging=0 --with-fc=0 --with-cmake --with-openmp=1 --with-mpi=1 --with-mpicc=mpicc && make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH all -j $(nproc) && make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH check

RUN cd $SLEPC_DIR && ./configure && make SLEPC_DIR=$SLEPC_DIR PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH all -j $(nproc) && make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH check


### real version
ENV PETSC_ARCH=petsc_real


RUN cd $PETSC_DIR && ./configure --with-scalar-type=real  --COPTFLAGS=$OPTFLAGS --CXXOPTFLAGS=$OPTFLAGS --FOPTFLAGS=$OPTFLAGS --with-64-bit-indices --with-debugging=0 --with-fc=0 --with-cmake --with-openmp=1 --with-mpi=1 --with-mpicc=mpicc && make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH all -j $(nproc) && make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH check 

RUN cd $SLEPC_DIR && ./configure   && make SLEPC_DIR=$SLEPC_DIR PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH all -j $(nproc) && make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH check




RUN chown -R user:user /opt/*

RUN chown -R user:user /home/user/


