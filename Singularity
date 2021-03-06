Bootstrap: docker
From: ubuntu:18.04

%labels
    Author wolfgang.hayek@niwa.co.nz

%help
    This container provides a complete LFRic build environment along with a "headless"
    version of ParaView (only ParaView Server and ParaView Python), as well as additional
    tools such as Iris (scitools.org.uk/iris) and essential Python packages.

%environment

    # PSyclone settings
    export PSYCLONE_CONFIG=/usr/local/share/psyclone/psyclone.cfg

    # Required for auto-loading the netCDFLFRicReader plugin
    export LD_LIBRARY_PATH=/usr/local/lib:/usr/local/lib/netCDFLFRicReader
    export PV_PLUGIN_PATH=/usr/local/lib/netCDFLFRicReader

    # LFRic build configs
    export FC=gfortran
    export FPP="cpp -traditional-cpp"
    export FFLAGS="-I/usr/local/include -I/usr/include/mpich -I/usr/include"
    export LDMPI=mpif90
    export LDFLAGS="-L/usr/local/lib -L/usr/lib/x86_64-linux-gnu/hdf5/mpich"

%post

    apt-get update && apt-get upgrade -y

    # Set timezone to UTC non-interactively
    ln -fs /usr/share/zoneinfo/UTC /etc/localtime
    apt-get install -y tzdata
    dpkg-reconfigure --frontend noninteractive tzdata

    apt-get install -y unzip xz-utils pkg-config git wget make cmake libmpich-dev libosmesa6-dev \
                       libtbb-dev libgeos-dev libproj-dev libhdf5-mpich-dev libudunits2-dev \
                       python3-dev python3-pip python3-setuptools \
                       g++ subversion liburi-perl m4 libcurl4-gnutls-dev

    # Make Python3 the default Python, required by pFUnit and LFRic build systems
    # Note that this may brake Python3-incompatible scripts
    update-alternatives --install /usr/local/bin/python python /usr/bin/python3 10
    update-alternatives --config python

    #
    # yaxt
    #

    yaxt_version=0.8.1
    yaxt_sha256=f258e289e143ee9e452885c2afbbd9279b9c51d01b0f4b886a67c88013dd9118

    # The configure script needs to run an MPI test executable which can fail if hostname is not recognised
    cp /etc/hosts /etc/hosts.bak
    hostname_current=$(hostname)
    echo "127.0.0.1 ${hostname_current}" > /etc/hosts
    echo "::1       ${hostname_current}" >> /etc/hosts

    wget https://www.dkrz.de/redmine/attachments/download/496/yaxt-${yaxt_version}.tar.gz
    echo "${yaxt_sha256} yaxt-${yaxt_version}.tar.gz" | sha256sum --check
    tar -xf yaxt-${yaxt_version}.tar.gz
    mkdir yaxt-${yaxt_version}_build
    cd yaxt-${yaxt_version}_build
    ../yaxt-${yaxt_version}/configure --with-idxtype=long CC=mpicc FC=mpif90 FPP="mpif90 -E -cpp"
    make -j 4
    make install
    cd ..
    rm -r yaxt-${yaxt_version}.tar.gz yaxt-${yaxt_version} yaxt-${yaxt_version}_build

    # Reset hosts file
    cat /etc/hosts.bak > /etc/hosts
    rm /etc/hosts.bak

    #
    # netCDF-C with parallel HDF5
    #

    ncc_version=4.7.4
    ncc_sha256=99930ad7b3c4c1a8e8831fb061cb02b2170fc8e5ccaeda733bd99c3b9d31666b

    wget https://github.com/Unidata/netcdf-c/archive/v${ncc_version}.tar.gz
    echo "${ncc_sha256} v${ncc_version}.tar.gz" | sha256sum --check
    tar -xf v${ncc_version}.tar.gz
    mkdir netcdf-c-${ncc_version}_build
    cd netcdf-c-${ncc_version}_build
    ../netcdf-c-${ncc_version}/configure CC=mpicc CXX=mpicxx FF=mpif90 FC=mpif90 CFLAGS="-I/usr/include/hdf5/mpich" \
      LDFLAGS="-L/usr/lib/x86_64-linux-gnu/hdf5/mpich -Wl,-rpath=/usr/lib/x86_64-linux-gnu/hdf5/mpich"
    make -j 4
    make install
    cd ..
    rm -r v${ncc_version}.tar.gz netcdf-c-${ncc_version} netcdf-c-${ncc_version}_build

    #
    # netCDF-Fortran with parallel HDF5
    #

    ncf_version=4.5.2
    ncf_sha256=0b05c629c70d6d224a3be28699c066bfdfeae477aea211fbf034d973a8309b49

    wget https://github.com/Unidata/netcdf-fortran/archive/v${ncf_version}.tar.gz
    echo "${ncf_sha256} v${ncf_version}.tar.gz" | sha256sum --check
    tar -xf v${ncf_version}.tar.gz
    mkdir netcdf-fortran-${ncf_version}_build
    cd netcdf-fortran-${ncf_version}_build
    ../netcdf-fortran-${ncf_version}/configure CC=mpicc CXX=mpicxx FF=mpif90 FC=mpif90
    make -j 4
    make install
    cd ..
    rm -r v${ncf_version}.tar.gz netcdf-fortran-${ncf_version} netcdf-fortran-${ncf_version}_build

    #
    # netCDF-C++ with parallel HDF5
    #

    nccxx_version=4.3.1
    nccxx_sha256=e3fe3d2ec06c1c2772555bf1208d220aab5fee186d04bd265219b0bc7a978edc

    wget https://github.com/Unidata/netcdf-cxx4/archive/v${nccxx_version}.tar.gz
    echo "${nccxx_sha256} v${nccxx_version}.tar.gz" | sha256sum --check
    tar -xf v${nccxx_version}.tar.gz
    mkdir netcdf-cxx4-${nccxx_version}_build
    cd netcdf-cxx4-${nccxx_version}_build
    ../netcdf-cxx4-${nccxx_version}/configure CC=mpicc CXX=mpicxx FF=mpif90 FC=mpif90 CFLAGS="-I/usr/include/hdf5/mpich" \
      LDFLAGS="-L/usr/local/lib -Wl,-rpath=/usr/local/lib"
    make -j 4
    make install
    cd ..
    rm -r v${nccxx_version}.tar.gz netcdf-cxx4-${nccxx_version} netcdf-cxx4-${nccxx_version}_build

    #
    # XIOS
    #

    xios_revision=1866

    ln -s /usr/bin/make /usr/bin/gmake
    svn co http://forge.ipsl.jussieu.fr/ioserver/svn/XIOS/trunk@${xios_revision} XIOS
    cd XIOS
    echo "export HDF5_INC_DIR=/usr/include/hdf5/mpich" > arch/arch-GCC_LINUX.env
    echo "export HDF5_LIB_DIR=/usr/lib/x86_64-linux-gnu/hdf5/mpich" >> arch/arch-GCC_LINUX.env
    echo "export NETCDF_INC_DIR=/usr/local/include" >> arch/arch-GCC_LINUX.env
    echo "export NETCDF_LIB_DIR=/usr/local/lib" >> arch/arch-GCC_LINUX.env
    ./make_xios --full --arch GCC_LINUX --job 4
    cp lib/libxios.a /usr/lib
    cp inc/* /usr/include
    cd ..
    rm -r XIOS

    #
    # pFUnit
    #

    pfunit_version=3.2.9
    pfunit_sha256=21c55f438b2ec7247645261e341fc03b2a07460becc6ec1c99c5ddb4aa971f70

    wget https://sourceforge.net/projects/pfunit/files/latest/download/pFUnit-${pfunit_version}.tgz
    echo "${pfunit_sha256} pFUnit-${pfunit_version}.tgz" | sha256sum --check
    tar -xf pFUnit-${pfunit_version}.tgz
    cd pFUnit-${pfunit_version}
    export F90_VENDOR=GNU
    export F90=gfortran
    export MPIF90=mpif90
    export INSTALL_DIR=/usr/local
    make
    make install
    cd ..
    rm -r pFUnit-${pfunit_version}.tgz pFUnit-${pfunit_version}

    #
    # Python packages
    #

    # Provides more recent versions than apt-get
    pip3 install cython matplotlib numpy pandas scipy Jinja2 PSyclone

    # Set PSyclone permissions to default for usr directory
    chmod 0755 /usr/local/bin/psyclone

    # Build these from source to make sure that netCDF/HDF5/GEOS system libraries are used
    pip3 install --no-binary :all: netCDF4 shapely cartopy

    # Pyke is required by Iris but not available through package managers
    wget https://sourceforge.net/projects/pyke/files/pyke/1.1.1/pyke3-1.1.1.zip
    echo "b877b390e70a2eacc01d97c3a992fde947276afc2798ca3ac6c6f74c796cb6dc pyke3-1.1.1.zip" | sha256sum --check
    unzip -q pyke3-1.1.1.zip
    cd pyke-1.1.1
    python3 setup.py install
    cd ..
    rm -r pyke-1.1.1 pyke3-1.1.1.zip

    # Install Iris with UGRID support
    pip3 install pyugrid scitools-iris

    #
    # ParaView
    #

    # Select ParaView major.minor version and patch to form "major.minor.patch"
    pv_version=5.8
    pv_patch=0
    pv_sha256=219e4107abf40317ce054408e9c3b22fb935d464238c1c00c0161f1c8697a3f9

    # Download and verity ParaView sources
    pv_url="https://www.paraview.org/paraview-downloads/download.php"
    pv_url_params="submit=Download&version=v${pv_version}&type=source&os=Sources&downloadFile="
    pv_filename=ParaView-v${pv_version}.${pv_patch}.tar.xz
    wget "${pv_url}?${pv_url_params}${pv_filename}" -O ${pv_filename}
    echo "${pv_sha256} ${pv_filename}" | sha256sum --check

    # Build ParaView without GUI and X ("headless") - this will provide ParaView Server
    # and ParaView Python. Use same netCDF/HDF5 libraries as Python-netCDF4 to avoid
    # runtime issues. Parallel NetCDF/HDF5 require using MPI C compiler.
    tar -xf ${pv_filename}
    mkdir -p ParaView-v${pv_version}.${pv_patch}/build
    cd ParaView-v${pv_version}.${pv_patch}/build
    cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    -DCMAKE_C_COMPILER=mpicc \
    -DCMAKE_CXX_COMPILER=mpiCC \
    -DCMAKE_Fortran_COMPILER=mpif90 \
    -DPARAVIEW_BUILD_EDITION=CANONICAL \
    -DPARAVIEW_BUILD_SHARED_LIBS=ON \
    -DPARAVIEW_INSTALL_DEVELOPMENT_FILES=ON \
    -DPARAVIEW_USE_QT=OFF \
    -DPARAVIEW_USE_PYTHON=ON \
    -DPARAVIEW_USE_MPI=ON \
    -DPARAVIEW_USE_VTKM=OFF \
    -DPARAVIEW_ENABLE_RAYTRACING=OFF \
    -DPARAVIEW_ENABLE_VISITBRIDGE=OFF \
    -DPARAVIEW_PLUGIN_ENABLE_NetCDFTimeAnnotationPlugin=ON \
    -DPARAVIEW_PLUGIN_ENABLE_StreamLinesRepresentation=ON \
    -DPARAVIEW_PLUGIN_ENABLE_StreamingParticles=ON \
    -DPARAVIEW_PLUGIN_ENABLE_SurfaceLIC=ON \
    -DVTK_USE_X=OFF \
    -DVTK_OPENGL_HAS_OSMESA=ON \
    -DVTK_SMP_IMPLEMENTATION_TYPE=tbb \
    -DVTK_MODULE_ENABLE_VTK_GeovisCore=YES \
    -DVTK_MODULE_ENABLE_VTK_IONetCDF=YES \
    -DVTK_MODULE_USE_EXTERNAL_VTK_netcdf=ON \
    -DVTK_MODULE_USE_EXTERNAL_VTK_hdf5=ON
    make -j 4
    make install
    cd ../..
    rm -r ParaView-v${pv_version}.${pv_patch} ${pv_filename}

    #
    # LFRic Reader plugin
    #

    git clone https://github.com/tinyendian/lfric_reader.git
    mkdir -p lfric_reader/src/cxx/build
    cd lfric_reader/src/cxx/build
    cmake .. -DBUILD_TESTING=OFF -DCMAKE_INSTALL_PREFIX=/usr/local \
             -DCMAKE_C_COMPILER=mpicc -DCMAKE_CXX_COMPILER=mpiCC
    make -j 4
    make install
    cd ../../../..
    rm -rf lfric_reader

    #
    # Catalyst adaptor
    #

    git clone https://github.com/tinyendian/catalyst_adaptor.git
    mkdir -p catalyst_adaptor/src/cxx/build
    cd catalyst_adaptor/src/cxx/build
    cmake .. -DBUILD_TESTING=OFF -DCMAKE_INSTALL_PREFIX=/usr/local \
             -DCMAKE_C_COMPILER=mpicc -DCMAKE_CXX_COMPILER=mpiCC
    make -j 4
    make install
    cd ../../../..
    rm -rf catalyst_adaptor
