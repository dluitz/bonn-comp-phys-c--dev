FROM gcc:15.1

## Additional packages
RUN echo "deb http://deb.debian.org/debian bookworm-backports main" >> /etc/apt/sources.list.d/debian-12-backports.list
RUN apt-get upgrade -y
RUN apt-get update -y
RUN apt-get install -y build-essential 
RUN apt-get update && apt-get install -y gfortran-12 \
    && ln -s /usr/bin/gfortran-12 /usr/bin/gfortran \
    && apt-get install -y libopenmpi-dev openmpi-bin openmpi-common 

# RUN apt-get install -y gfortran
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
RUN apt-get install -y libopenmpi-dev openmpi-bin openmpi-common 
RUN git clone https://github.com/dmtcp/dmtcp.git /opt/dmtcp && \
    cd /opt/dmtcp && \
    ./configure && \
    make -j$(nproc) && \
    make install

RUN apt-get install petsc-dev -y

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


ENV LIBS=/opt
ENV OPTFLAGS="-O3"
ENV PETSC_DIR=$LIBS/petsc
ENV SLEPC_DIR=$LIBS/slepc
ENV PETSC_ARCH=complex

ENV OMPI_ALLOW_RUN_AS_ROOT=1
ENV OMPI_ALLOW_RUN_AS_ROOT_CONFIRM=1

# Clone PETSc
RUN mkdir -p $LIBS && \
    git clone -b release https://gitlab.com/petsc/petsc.git $PETSC_DIR && \
    cd $PETSC_DIR && git checkout v3.23.4

# Clone SLEPc
RUN git clone https://gitlab.com/slepc/slepc $SLEPC_DIR && cd $SLEPC_DIR && git checkout v3.23.0


# Build PETSc (complex)
WORKDIR $PETSC_DIR
RUN ./configure --with-scalar-type=complex \
                --COPTFLAGS="$OPTFLAGS" \
                --CXXOPTFLAGS="$OPTFLAGS" \
                --FOPTFLAGS="$OPTFLAGS" \
                --with-64-bit-indices \
                --with-debugging=0 \
                --with-fc=0 \
                --with-cmake \
                --with-openmp=1 \
                --with-mpi=1 && \
    make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH all -j$(nproc) 
    # make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH check


# Build SLEPc (complex)
WORKDIR $SLEPC_DIR
RUN ./configure  && make SLEPC_DIR=$SLEPC_DIR PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH all -j$(nproc) 


ENV PETSC_ARCH=real


# Build PETSc (complex)
WORKDIR $PETSC_DIR
RUN ./configure --with-scalar-type=real \
                --COPTFLAGS="$OPTFLAGS" \
                --CXXOPTFLAGS="$OPTFLAGS" \
                --FOPTFLAGS="$OPTFLAGS" \
                --with-64-bit-indices \
                --with-debugging=0 \
                --with-fc=0 \
                --with-cmake \
                --with-openmp=1 \
                --with-mpi=1 && \
    make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH all -j$(nproc) 
    # make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH check


# Build SLEPc (complex)
WORKDIR $SLEPC_DIR
RUN ./configure  && make SLEPC_DIR=$SLEPC_DIR PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH all -j$(nproc) 


# make SLEPC_DIR=$SLEPC_DIR PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH check

# # Build PETSc (real)
# WORKDIR $PETSC_DIR
# RUN ./configure --with-scalar-type=real \
#                 --COPTFLAGS="$OPTFLAGS" \
#                 --CXXOPTFLAGS="$OPTFLAGS" \
#                 --FOPTFLAGS="$OPTFLAGS" \
#                 --with-64-bit-indices \
#                 --with-debugging=0 \
#                 --with-fc=0 \
#                 --with-cmake \
#                 --with-openmp=1 \
#                 --with-mpi=1 \
#                 --prefix=$PETSC_DIR/real && \
#     make PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_real all -j$(nproc) && \
#     make PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_real check
#
# # Build SLEPc (real)
# WORKDIR $SLEPC_DIR
# RUN ./configure --prefix=$SLEPC_DIR/real && \
#     make SLEPC_DIR=$SLEPC_DIR PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_real all -j$(nproc) && \
#     make SLEPC_DIR=$SLEPC_DIR PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_real check
#


RUN chown -R user:user /opt/*
RUN chown -R user:user /home/user/
USER user
