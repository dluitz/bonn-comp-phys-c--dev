FROM gcc:latest

# Set environment variables
ENV LIBS=/opt
ENV OPTFLAGS="-O3"
ENV PETSC_DIR=$LIBS/petsc
ENV SLEPC_DIR=$LIBS/slepc

# RUN echo "deb http://deb.debian.org/debian bookworm-backports main" >> /etc/apt/sources.list.d/debian-12-backports.list

RUN apt-get upgrade -y
RUN apt-get update -y


RUN apt-get install gdb -y
RUN apt-get install gcc -y
RUN apt-get install g++ -y
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
RUN apt-get install build-essential autoconf automake libtool -y
# Install system packages
# RUN apt-get update && apt-get install -y --no-install-recommends \
#     gdb valgrind vim neovim git cmake wget curl \
#     libhdf5-dev h5utils hdf5-tools libgsl-dev libcatch2-dev parallel \
#     python3-numpy python3-scipy python3-matplotlib python3-h5py \
#     openmpi-bin libopenmpi-dev libc++-19-dev libomp-19-dev 
# RUN apt-get update && apt-get install -y --no-install-recommends \
#     build-essential autoconf automake libtool gdb


USER root

# Install DMTCP
RUN which gcc
RUN which g++
RUN mkdir -p $LIBS/dmtcp
RUN echo $LIBS/dmtcp
RUN git clone https://github.com/dmtcp/dmtcp.git $LIBS/dmtcp 
RUN chmod -R 777 $LIBS/dmtcp
WORKDIR $LIBS/dmtcp  
RUN ./configure --verbose 
RUN make 
RUN make install

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

# Fix permissions
USER root
RUN chown -R user:user $LIBS && chown -R user:user /home/user

# Create non-root user for MPI
RUN useradd -ms /bin/bash user
USER user
WORKDIR /home/user
