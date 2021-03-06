FROM centos:latest
MAINTAINER John Halley Gotway <johnhg@ucar.edu>

# 
# This Dockerfile compiles MET from source during "docker build" step
#
ENV MET_VERSION 6.0 
ENV MET_RELEASE_DATE 20170403

#
# Compilers
#
ENV CC  /usr/bin/gcc
ENV CXX /usr/bin/g++
ENV FC  /usr/bin/gfortran
ENV F77 /usr/bin/gfortran

#
# Package URL's
#
ENV HDF4_URL       http://www.hdfgroup.org/ftp/HDF/releases/HDF4.2r3/src/HDF4.2r3.tar.gz
ENV HDFEOS_URL     ftp://edhs1.gsfc.nasa.gov/edhs/hdfeos/previous_releases/HDF-EOS2.16v1.00.tar.Z

ENV NETCDF4C_URL   https://github.com/Unidata/netcdf-c/archive/v4.4.1.1.zip
ENV NETCDF4CXX_URL https://github.com/Unidata/netcdf-cxx4/archive/v4.3.0.tar.gz

ENV BUFRLIB_URL    http://www.dtcenter.org/met/users/support/online_tutorial/METv5.0/compilation/tar_files/BUFRLIB_v10-2-3.tar
ENV GSFONT_URL     http://www.dtcenter.org/met/users/support/online_tutorial/docker_data/ghostscript-fonts-std-8.11.tar.gz

ENV MET_URL        http://www.dtcenter.org/met/users/downloads/MET_releases/met-${MET_VERSION}.${MET_RELEASE_DATE}.tar.gz
ENV PATCH_URL      http://www.dtcenter.org/met/users/support/known_issues/METv${MET_VERSION}/patches/met-${MET_VERSION}_patches_latest.tar.gz
ENV DATA_URL       http://www.dtcenter.org/met/users/support/online_tutorial/tutorial_data/METv${MET_VERSION}_tutorial_data.tar.gz

#
# Install required packages
#
RUN yum -y update \
 && yum -y install file gcc gcc-gfortran gcc-c++ glibc.i686 libgcc.i686 \
                   libpng-devel jasper jasper-devel zlib zlib-devel \
                   cairo-devel freetype-devel epel-release \
                   hostname m4 make tar tcsh time wget which \
                   flex flex-devel bison bison-devel unzip \
 && yum -y install g2clib-devel hdf5-devel.x86_64 gsl-devel \
 && yum -y install gv ncview wgrib wgrib2 ImageMagick ps2pdf

#
# Set working directory
#
WORKDIR /met

#
# Environment for interactive bash and csh container shells
#
RUN echo export MET_FONT_DIR=/met/external_libs/fonts >> /etc/bashrc \
 && echo setenv MET_FONT_DIR /met/external_libs/fonts >> /etc/csh.cshrc \
 && echo export LD_LIBRARY_PATH=/usr/local/lib >> /etc/bashrc \
 && echo setenv LD_LIBRARY_PATH /usr/local/lib >> /etc/csh.cshrc

ENV LD_LIBRARY_PATH /usr/local/lib

#
# Download BUFRLIB and GhostScript fonts
#
RUN mkdir -p /met/external_libs/BUFRLIB \
 && cd /met/external_libs/BUFRLIB \
 && echo "Downloading BUFRLIB from ${BUFRLIB_URL}" \
 && curl -SL ${BUFRLIB_URL} | tar xC /met/external_libs/BUFRLIB \
 && cat preproc.sh | sed 's/cpp /cpp -traditional-cpp /g' > preproc_patch.sh \
 && chmod +x preproc_patch.sh \
 && LOG_FILE=/met/external_libs/BUFRLIB/build.log \
 && echo "Compiling BUFRLIB and writing log file ${LOG_FILE}" \
 && ./preproc_patch.sh *.F > ${LOG_FILE} \
 && ${CC} -c -DUNDERSCORE *.c >> ${LOG_FILE} \
 && ${FC} -c -fno-second-underscore *.f >> ${LOG_FILE} \
 && ar crv libbufr.a *.o >> ${LOG_FILE} \
 && rm -f /usr/lib/libbufr.a \
 && cp  *.a /usr/lib \
 && echo "Downloading GhostScript fonts from ${GSFONT_URL}" \
 && curl -SL ${GSFONT_URL} | tar zxC /met/external_libs

#
# NetCDF4 (c and c++)
#
RUN mkdir -p /met/external_libs \
 && cd /met/external_libs \
 && echo "Downloading netcdf-c-4.4.1.1 from ${NETCDF4C_URL}" \
 && wget ${NETCDF4C_URL} \
 && unzip v4.4.1.1.zip \
 && cd netcdf-c-4.4.1.1 \
 && LOG_FILE=/met/external_libs/netcdf-c-4.4.1.1/configure.log \
 && echo "Configuring netcdf-c-4.4.1.1 and writing log file ${LOG_FILE}" \
 && ./configure > ${LOG_FILE} \
 && LOG_FILE=/met/external_libs/netcdf-c-4.4.1.1/make_install.log \
 && echo "Compiling netcdf-c-4.4.1.1 and writing log file ${LOG_FILE}" \
 && make install > ${LOG_FILE} \
 && echo "Downloading  from ${NETCDF4CXX_URL}" \
 && cd /met/external_libs \
 && wget ${NETCDF4CXX_URL} \
 && tar -xzf v4.3.0.tar.gz \
 && cd netcdf-cxx4-4.3.0 \
 && LOG_FILE=/met/external_libs/netcdf-cxx4-4.3.0/configure.log \
 && echo "Configuring netcdf-cxx4-4.3.0 and writing log file ${LOG_FILE}" \
 && ./configure > ${LOG_FILE} \
 && LOG_FILE=/met/external_libs/netcdf-cxx4-4.3.0/make_install.log \
 && echo "Compiling netcdf-cxx4-4.3.0 and writing log file ${LOG_FILE}" \
 && make install > ${LOG_FILE}

#
# Download HDF4 and HDFEOS
#
RUN echo "Downloading HDF4.2r3 from ${HDF4_URL}" \
 && curl -SL ${HDF4_URL} | tar zxC /met/external_libs \
 && cd /met/external_libs/HDF4.2r3 \
 && LOG_FILE=/met/external_libs/HDF4.2r3/configure.log \
 && echo "Configuring HDF4.2r3 and writing log file ${LOG_FILE}" \
 && ./configure --prefix=/met/external_libs/HDF4.2r3 --disable-netcdf > ${LOG_FILE} \
 && cat mfhdf/hdiff/Makefile | sed 's/LIBS = -ljpeg -lz/LIBS = -ljpeg -lz -lm/g' > Makefile_NEW \
 && mv -f Makefile_NEW mfhdf/hdiff/Makefile \
 && LOG_FILE=/met/external_libs/HDF4.2r3/make_install.log \
 && echo "Compiling HDF4.2r3 and writing log file ${LOG_FILE}" \
 && make install > ${LOG_FILE} \
 && echo "Downloading hdfeos from ${HDFEOS_URL}" \
 && curl -SL ${HDFEOS_URL} | tar zxC /met/external_libs \
 && cd /met/external_libs/hdfeos \
 && LOG_FILE=/met/external_libs/hdfeos/configure.log \
 && echo "Configuring hdfeos and writing log file ${LOG_FILE}" \
 && ./configure --prefix=/met/external_libs/hdfeos --with-hdf4=/met/external_libs/HDF4.2r3 CC=/met/external_libs/HDF4.2r3/bin/h4cc > ${LOG_FILE} \
 && LOG_FILE=/met/external_libs/hdfeos/make_install.log \
 && echo "Compiling hdfeos and writing log file ${LOG_FILE}" \
 && make install > ${LOG_FILE}

#
# Download and compile MET source code and patches
#
RUN echo "Downloading met-${MET_VERSION} from ${MET_URL}" \
 && curl -SL ${MET_URL} | tar zxC /met \
 && echo "Downloading met-${MET_VERSION} patches from ${PATCH_URL}" \
 && curl -SL ${PATCH_URL} | tar zxC /met/met-${MET_VERSION} \
 && cd /met/met-${MET_VERSION} \
 && LOG_FILE=/met/met-${MET_VERSION}/configure.log \
 && echo "Configuring met-${MET_VERSION} and writing log file ${LOG_FILE}" \
 && ./configure --enable-grib2 --enable-mode_graphics --enable-modis --enable-lidar2nc \
    MET_HDF=/met/external_libs/HDF4.2r3 \
    MET_HDFEOS=/met/external_libs/hdfeos \
    MET_FREETYPEINC=/usr/include/freetype2 MET_FREETYPELIB=/usr/lib \
    MET_CAIROINC=/usr/include/cairo MET_CAIROLIB=/usr/lib > ${LOG_FILE} \
 && LOG_FILE=/met/met-${MET_VERSION}/make_install.log \
 && echo "Compiling met-${MET_VERSION} and writing log file ${LOG_FILE}" \
 && make install > ${LOG_FILE} \
 && LOG_FILE=/met/met-${MET_VERSION}/make_test.log \
 && echo "Testing met-${MET_VERSION} and writing log file ${LOG_FILE}" \
 && make test > ${LOG_FILE} 2>&1

#
# Pull input data for the online tutorial
#
RUN echo "Downloading input data for the met-${MET_VERSION} online tutorial from ${DATA_URL}" \
 && curl -SL ${DATA_URL} | tar zxC /met/met-${MET_VERSION} data

