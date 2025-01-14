# Process this file with autoconf to produce a configure script.
# author: Mike L. Smith (grimbough@gmail.com)

AC_PREREQ([2.65])
AC_INIT([Rhdf5lib], [1.13], [grimbough@gmail.com])
AC_CONFIG_SRCDIR([src])

m4_include([m4/ax_check_compile_flag.m4])

# Check the compiler configured with R
: ${R_HOME=`R RHOME`}
if test -z "${R_HOME}"; then
  echo "could not determine R_HOME"
  exit 1
fi

CC=`"${R_HOME}/bin/R" CMD config CC`
CFLAGS=`"${R_HOME}/bin/R" CMD config CFLAGS`
CPICFLAGS=`"${R_HOME}/bin/R" CMD config CPICFLAGS`
CXX=`"${R_HOME}/bin/R" CMD config CXX`
CXXFLAGS=`"${R_HOME}/bin/R" CMD config CXXFLAGS`
CXXPICFLAGS=`"${R_HOME}/bin/R" CMD config CXXPICFLAGS`
CPPFLAGS=`"${R_HOME}/bin/R" CMD config CPPFLAGS`
F77=`"${R_HOME}/bin/R" CMD config F77`
MAKE=`"${R_HOME}/bin/R" CMD config MAKE`
LDFLAGS=`"${R_HOME}/bin/R" CMD config LDFLAGS`
AC_LANG(C++)

dnl Checks for programs.
AC_PROG_CC
AC_PROG_CXX

CC_BASENAME="`echo $CC | cut -f1 -d' '`"

dnl hdf5 spews loads of errors.  Use -w to suppress these if supported
#AX_CHECK_COMPILE_FLAG([-w], [CFLAGS="$CFLAGS -w"; CXXFLAGS="$CXXFLAGS -w"])

dnl specifically suppress warnings R decides are "Significant" and produces warnings in R CMD check
if test "X$CC_BASENAME" = "Xgcc"; then
  echo "COMPILER = $CC_BASENAME"
  ## we supress C warnings that trigger a WARNING from R CMD check 
  ## we aren't going to edit the HDF5 source code much
  AX_CHECK_COMPILE_FLAG([-Wno-unused-variable -Wno-unused-parameter], 
                        [CFLAGS="$CFLAGS -Wno-unused-variable -Wno-unused-parameter"])
  AX_CHECK_COMPILE_FLAG([-Wno-cast-qual], 
                        [CFLAGS="$CFLAGS -Wno-cast-qual"])
  AX_CHECK_COMPILE_FLAG([-Wno-pedantic -Wno-extra], 
                        [CFLAGS="$CFLAGS -Wno-pedantic -Wno-extra"])
  AX_CHECK_COMPILE_FLAG([-Wno-format-overflow -Wno-format-truncation], 
                        [CFLAGS="$CFLAGS -Wno-format-overflow -Wno-format-truncation"])
  AX_CHECK_COMPILE_FLAG([-Wno-stringop-overflow -Wno-overlength-strings -Wno-stringop-truncation], 
                        [CFLAGS="$CFLAGS -Wno-stringop-overflow -Wno-overlength-strings -Wno-stringop-truncation"])
fi

if test "X$CC_BASENAME" = "Xclang"; then
  echo "COMPILER = $CC_BASENAME"
  AX_CHECK_COMPILE_FLAG([-Wno-self-assign], 
                        [CFLAGS="$CFLAGS -Wno-self-assign"]) 
  AX_CHECK_COMPILE_FLAG([-Wno-bitwise-op-parentheses], 
                        [CFLAGS="$CFLAGS -Wno-bitwise-op-parentheses"])
  AX_CHECK_COMPILE_FLAG([-Wno-cast-align -Wno-cast-qual], 
                        [CFLAGS="$CFLAGS -Wno-cast-align -Wno-cast-qual"])
  AX_CHECK_COMPILE_FLAG([-Wno-double-promotion], 
                        [CFLAGS="$CFLAGS -Wno-double-promotion"])
  AX_CHECK_COMPILE_FLAG([-Wno-implicit-fallthrough], 
                        [CFLAGS="$CFLAGS -Wno-implicit-fallthrough"])
  AX_CHECK_COMPILE_FLAG([-Wno-sometimes-uninitialized], 
                        [CFLAGS="$CFLAGS -Wno-sometimes-uninitialized"])
fi

### 'checking for zlib... ';
AC_ARG_WITH([zlib],
            [AS_HELP_STRING([--with-zlib=DIR],
                            [Use zlib library for external deflate I/O
                             filter [default=yes]])],
            [],
            [withval=yes])

case $withval in
  yes)
    HAVE_ZLIB="yes"
    AC_CHECK_HEADERS([zlib.h], [ZLIB_HOME="yes"], [unset HAVE_ZLIB], [AC_INCLUDES_DEFAULT $CFLAGS $CXXFLAGS])
    if test -z "$HAVE_ZLIB"; then
        AC_MSG_ERROR([couldn't find zlib library.  
        Please specify a location using --with-zlib=/path/to/zlib])
    fi
    ZLIB_LIB=""
    ZLIB_INCLUDE=""
    ;;
  no)
    AC_MSG_ERROR([zlib is required for Rhdf5lib])
    ;;
  *)
    HAVE_ZLIB="yes"
    ## return FALSE if $withval is empty
    if test -n "$withval"; then
      ZLIB_HOME="$withval"
      if test ! -f "${ZLIB_HOME}/include/zlib.h"; then
        AC_MSG_ERROR([zlib is required but not found in ${ZLIB_HOME}])
      fi
    else
      AC_MSG_ERROR([zlib is required but no path specified])
    fi
    ZLIB_LIB="-L${ZLIB_HOME}/lib/"
    ZLIB_INCLUDE="-I${ZLIB_HOME}/include/"
    ;;
esac

#echo ${CPPFLAGS}
#echo ${LDFLAGS}
dnl Trying to ensure OpenSSL is found on BioC Mac builder
dnl Based on similar implementation found in rtracklayer
#case "$OSTYPE" in "darwin"*)
#  BREWDIR=`brew --prefix`
#  export PKG_CONFIG_PATH=$BREWDIR/opt/openssl/lib/pkgconfig
#  ;;
#esac
#
#PKG_CHECK_MODULES([OPENSSL], [openssl >= 1.0], [OPENSSL="yes"], [OPENSSL="no"])
#if test "${OPENSSL}" != "no"; then
#  CPPFLAGS="${CPPFLAGS} ${OPENSSL_CFLAGS}"
#  LDFLAGS="${LDFLAGS} ${OPENSSL_LIBS}"
#fi
#echo ${CPPFLAGS}
#echo ${LDFLAGS}

ROS3_VFD=yes
if test "X$ROS3_VFD" = "Xyes"; then
    AC_CHECK_HEADERS([curl/curl.h],, [unset ROS3_VFD])
    AC_CHECK_HEADERS([openssl/evp.h],, [unset ROS3_VFD])
    AC_CHECK_HEADERS([openssl/hmac.h],, [unset ROS3_VFD])
    AC_CHECK_HEADERS([openssl/sha.h],, [unset ROS3_VFD])
    if test "X$ROS3_VFD" = "Xyes"; then
        AC_CHECK_LIB([curl], [curl_global_init],, [unset ROS3_VFD])
        AC_CHECK_LIB([crypto], [EVP_sha256],, [unset ROS3_VFD])
    fi
    if test "X$ROS3_VFD" = "Xyes"; then
        WITH_S3_VFD="--enable-ros3-vfd"
        CURL_LIBS="-lcrypto -lcurl"
    else
        WITH_S3_VFD="--enable-ros3-vfd=no"
        CURL_LIBS=""
    fi
fi
echo "S3_VFD=${WITH_S3_VFD}"

#untar the lib
BASEPBNAME="hdf5"
PBTGZNAME=hdf5small_cxx_hl_1.10.7.tar.gz
cd src
if test -d ./${BASEPBNAME}; then 
	echo 'found ' $BASEPBNAME ' header sources and tar archive;using what is there.'
else
	echo "untarring $PBTGZNAME ...";
	gunzip -dc ${PBTGZNAME} | tar xf -;
fi;

echo "building the szip library...";
cd ${BASEPBNAME}/szip;
./configure --with-pic --enable-shared=no \
    CC="${CC}" CFLAGS="${CFLAGS} ${CPICFLAGS}" CPPFLAGS="${CPPFLAGS}"
$MAKE
$MAKE install
SZIP_HOME=`pwd -P`/szip


echo "building the hdf5 library...";
cd ../;
./configure --with-pic --enable-shared=no --enable-cxx --enable-hl \
    --with-szlib=${SZIP_HOME} --with-zlib=${ZLIB_HOME} \
    ${WITH_S3_VFD} \
    CXX="${CXX}" CXXFLAGS="${CXXFLAGS} ${CXXPICFLAGS}" \
    CC="${CC}" CFLAGS="${CFLAGS} ${CPICFLAGS}" \
    CPPFLAGS="${CPPFLAGS}" LDFLAGS="${LDFLAGS}"
$MAKE lib
#echo "Make return value: $?"
AS_IF([ test "$?" != 0 ], 
      [AC_MSG_ERROR("Failed to compiled the HDF5 library")], 
      [echo test passed])
cd ../../

#update variables in Makevars
HDF5_INCLUDE="${BASEPBNAME}/src"
HDF5_CXX_INCLUDE="${BASEPBNAME}/c++/src"
HDF5_HL_INCLUDE="${BASEPBNAME}/hl/src"
HDF5_HL_CXX_INCLUDE="${BASEPBNAME}/hl/c++/src"
AC_SUBST(HDF5_INCLUDE)
AC_SUBST(HDF5_CXX_INCLUDE)
AC_SUBST(HDF5_HL_INCLUDE)
AC_SUBST(HDF5_HL_CXX_INCLUDE)
AC_MSG_NOTICE([   HDF5_INCLUDE=${HDF5_INCLUDE}])
AC_MSG_NOTICE([   HDF5_CXX_INCLUDE=${HDF5_CXX_INCLUDE}])
AC_MSG_NOTICE([   HDF5_HL_INCLUDE=${HDF5_HL_INCLUDE}])
AC_MSG_NOTICE([   HDF5_HL_CXX_INCLUDE=${HDF5_HL_CXX_INCLUDE}])

HDF5_LIB="${BASEPBNAME}/src/.libs/libhdf5.a"
HDF5_CXX_LIB="${BASEPBNAME}/c++/src/.libs/libhdf5_cpp.a"
HDF5_HL_LIB="${BASEPBNAME}/hl/src/.libs/libhdf5_hl.a"
HDF5_HL_CXX_LIB="${BASEPBNAME}/hl/c++/src/.libs/libhdf5_hl_cpp.a"
SZIP_LIB="${BASEPBNAME}/szip/src/.libs/libsz.a"

AC_SUBST(HDF5_LIB)
AC_SUBST(HDF5_CXX_LIB)
AC_SUBST(HDF5_HL_LIB)
AC_SUBST(HDF5_HL_CXX_LIB)
AC_SUBST(SZIP_LIB)
AC_SUBST(ZLIB_HOME)
AC_SUBST(ZLIB_LIB)
AC_SUBST(ZLIB_INCLUDE)

AC_MSG_NOTICE([   HDF5_LIB=${HDF5_LIB}])
AC_MSG_NOTICE([   HDF5_CXX_LIB=${HDF5_CXX_LIB}])
AC_MSG_NOTICE([   HDF5_HL_LIB=${HDF5_HL_LIB}])
AC_MSG_NOTICE([   HDF5_HL_CXX_LIB=${HDF5_HL_CXX_LIB}])
AC_MSG_NOTICE([   SZIP_LIB=${SZIP_LIB}])

AC_CONFIG_FILES([src/Makevars])

AC_OUTPUT
