FROM gcc:latest

# Set environment variables
ENV LIBS=/opt
ENV OPTFLAGS="-O3"
ENV PETSC_DIR=$LIBS/petsc
ENV SLEPC_DIR=$LIBS/slepc

# Install system packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    gdb valgrind vim neovim git cmake wget curl \
    libhdf5-dev h5utils hdf5-tools libgsl-dev libcatch2-dev parallel \
    python3-numpy python3-scipy python3-matplotlib python3-h5py \
    openmpi-bin libopenmpi-dev libc++-19-dev libomp-19-dev && \
    rm -rf /var/lib/apt/lists/*

# Create non-root user for MPI
RUN useradd -ms /bin/bash user
USER user
WORKDIR /home/user

# Install DMTCP
RUN git clone https://github.com/dmtcp/dmtcp.git $LIBS/dmtcp && \
    cd $LIBS/dmtcp && ./configure && make -j$(nproc) && make install

# Install Eigen
RUN mkdir -p $LIBS/Eigen && \
    git clone https://gitlab.com/libeigen/eigen.git $LIBS/Eigen/eigen && \
    mkdir $LIBS/Eigen/build && \
    cd $LIBS/Eigen/build && \
    cmake ../eigen -DCMAKE_INSTALL_PREFIX=$LIBS/Eigen/install -DINCLUDE_INSTALL_DIR=$LIBS/Eigen/install/include && \
    make install && \
    rm -rf $LIBS/Eigen/eigen

# Install HighFive
RUN mkdir -p $LIBS/HighFive && \
    git clone --recursive https://github.com/highfive-devs/highfive.git $LIBS/HighFive/highfive && \
    mkdir $LIBS/HighFive/highfive/build && \
    cd $LIBS/HighFive/highfive/build && \
    cmake .. -DCMAKE_INSTALL_PREFIX=$LIBS/HighFive/install && \
    make -j$(nproc) && make install && \
    rm -rf $LIBS/HighFive/highfive

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
                --with-mpi=1 \
                --with-mpicc=mpicc \
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
                --with-mpi=1 \
                --with-mpicc=mpicc \
                --prefix=$PETSC_DIR/real && \
    make PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_real all -j$(nproc) && \
    make PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_real check

# Build SLEPc (real)
WORKDIR $SLEPC_DIR
RUN ./configure --prefix=$SLEPC_DIR/real && \
    make SLEPC_DIR=$SLEPC_DIR PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_real all -j$(nproc) && \
    make SLEPC_DIR=$SLEPC_DIR PETSC_DIR=$PETSC_DIR PETSC_ARCH=petsc_real check

# Fix permissions
USER root
RUN chown -R user:user $LIBS && chown -R user:user /home/user
