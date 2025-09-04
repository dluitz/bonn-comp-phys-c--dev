FROM gcc:15.1

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

# Clone PETSc
RUN mkdir -p $LIBS && \
    git clone -b release https://gitlab.com/petsc/petsc.git $PETSC_DIR && \
    cd $PETSC_DIR && git checkout v3.23.4

# Clone SLEPc
RUN git clone https://gitlab.com/slepc/slepc $SLEPC_DIR && \
    cd $SLEPC_DIR && git checkout v3.23.0

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
                --download-mpi \
                --prefix=$PETSC_DIR/complex && \
    make PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_complex all -j$(nproc) && \
    make PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_complex check

# Build SLEPc (complex)
WORKDIR $SLEPC_DIR
RUN ./configure --prefix=$SLEPC_DIR/complex && \
    make SLEPC_DIR=$SLEPC_DIR PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_complex all -j$(nproc) && \
    make SLEPC_DIR=$SLEPC_DIR PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_complex check

# Build PETSc (real)
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
                --download-mpi \
                --prefix=$PETSC_DIR/real && \
    make PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_real all -j$(nproc) && \
    make PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_real check

# Build SLEPc (real)
WORKDIR $SLEPC_DIR
RUN ./configure --prefix=$SLEPC_DIR/real && \
    make SLEPC_DIR=$SLEPC_DIR PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_real all -j$(nproc) && \
    make SLEPC_DIR=$SLEPC_DIR PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_real check





RUN chown -R user:user /opt/*

RUN chown -R user:user /home/user/
