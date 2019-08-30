BootStrap: shub
From: willgpaik/centos7_aci

%setup

%files

%environment 
    export PATH=$PATH:/opt/sw/dssp/bin
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
    source /usr/local/gromacs/bin/GMXRC

%runscript

%post
    yum -y update
    
    source /opt/rh/devtoolset-8/enable
    
    export BOOST_ROOT=/usr/local
    
    mkdir -p /opt/sw/dssp
    cd /opt/sw
    git clone https://github.com/mhekkel/libzeep.git
    cd libzeep
    
    sed -i -e '23s/-g/-g $(BOOST_LIB_DIR:%=-L%)/g' makefile
    sed -i -e 's|# BUILD_SHARED_LIB = $(BUILD_SHARED_LIB)|BUILD_SHARED_LIB = 1|g' makefile
    sed -i -e 's|# BOOST = $(HOME)/my-boost|BOOST = $(BOOST_ROOT)|g' makefile
    sed -i -e 's|BOOST_LIB_DIR       = $(BOOST:%=%/lib)|BOOST_LIB_DIR       = $(BOOST_ROOT)/lib|g' makefile
    sed -i -e 's|BOOST_INC_DIR       = $(BOOST:%=%/include)|BOOST_INC_DIR       = $(BOOST_ROOT)/include|g' makefile
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$BOOST_ROOT/lib:/opt/sw/libzeep/lib:/lib:/lib64
    export CPATH=$CPATH:$BOOST_ROOT/include:/opt/sw/libzeep/include/zeep
    
    make lib -j 2
    
    cd /opt/sw
    
    # Install dssp
    git clone https://github.com/cmbi/xssp.git
    cd xssp
    ./autogen.sh
    ./configure --prefix=/opt/sw/dssp/ --with-boost=$BOOST_ROOT/ --with-boost-libdir=$BOOST_ROOT/lib \
        CPPFLAGS="-I${BOOST_ROOT}/include -I${BUILD_DIR}/include -I/opt/sw/libzeep/include/zeep" \
        LDFLAGS="-L${BOOST_ROOT}/lib -L${BUILD_DIR}/lib -L/opt/sw/libzeep/lib" \
        CFLAGS=-lrt CXXFLAGS=-lrt
    make -j $NP && make install
    
    # Install GROMACS
    cd /opt/sw
    wget http://ftp.gromacs.org/pub/gromacs/gromacs-2019.3.tar.gz
    tar xfz gromacs-2019.3.tar.gz
    cd gromacs-2019.3
    mkdir build
    cd build
    cmake3 .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON
    make
    make check
    make install
    source /usr/local/gromacs/bin/GMXRC
    
    cd /opt/sw
    rm -rf xssp
    rm -rf gromacs-2019.3*
    
    
