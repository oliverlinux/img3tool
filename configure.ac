AC_PREREQ([2.69])
AC_INIT([img3tool], m4_esyscmd([git rev-list --count HEAD | tr -d '\n']), [tihmstar@gmail.com])

AC_CANONICAL_SYSTEM
AC_CANONICAL_HOST
AM_PROG_LIBTOOL

AM_INIT_AUTOMAKE([subdir-objects])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIRS([m4])


AC_DEFINE([VERSION_COMMIT_COUNT], "m4_esyscmd([git rev-list --count HEAD | tr -d '\n'])", [Git commit count])
AC_DEFINE([VERSION_COMMIT_SHA], "m4_esyscmd([git rev-parse HEAD | tr -d '\n'])", [Git commit sha])
AC_SUBST([VERSION_COMMIT_COUNT], ["m4_esyscmd([git rev-list --count HEAD | tr -d '\n'])"])
AC_SUBST([VERSION_COMMIT_SHA], ["m4_esyscmd([git rev-parse HEAD | tr -d '\n'])"])

# Checks for programs.
AC_PROG_CXX
AC_PROG_CC

crypto_backend="none"
# Check for operating system
AC_MSG_CHECKING([whether we need platform-specific build settings])
case $host_os in
  darwin* )
  AC_DEFINE(HAVE_COMMCRYPTO, 1, [Define if you have CommonCrypto (macOS)])
  AC_MSG_RESULT([${host_os}])
  crypto_backend="CommCrypto"
        ;;
  *mingw32*|*cygwin*)
  AC_MSG_RESULT([${host_os}])
  win32=true
  LDFLAGS+=" -no-undefined -lws2_32"
      ;;
esac

CXXFLAGS+=" -std=c++11"
CFLAGS+=" -std=c11"
# Versioning.

# Checks for libraries.
OPENSSL_REQUIRES_STR="openssl >= 0.9.8"
LIBPLIST_REQUIRES_STR="libplist-2.0 >= 2.3.0"
LIBGENERAL_REQUIRES_STR="libgeneral >= 65"
PKG_CHECK_MODULES(openssl, $OPENSSL_REQUIRES_STR, have_openssl=yes, have_openssl=no)
PKG_CHECK_MODULES(libplist, $LIBPLIST_REQUIRES_STR, have_plist=yes, have_plist=no)
PKG_CHECK_MODULES(libgeneral, $LIBGENERAL_REQUIRES_STR)

AC_SUBST([libgeneral_requires], [$LIBGENERAL_REQUIRES_STR])

AC_ARG_ENABLE([debug],
            [AS_HELP_STRING([--enable-debug],
            [enable debug build(default is no)])],
            [debug_build=true],
            [debug_build=false])

AC_ARG_WITH([plist],
            [AS_HELP_STRING([--without-plist],
            [do not build with libplist @<:@default=yes@:>@])],
            [with_plist=no],
            [with_plist=yes])

AC_ARG_WITH([openssl],
            [AS_HELP_STRING([--without-openssl],
            [do not build with openssl @<:@default=yes@:>@])],
            [with_openssl=no],
            [with_openssl=yes])

if test "x$with_plist" == "xyes"; then
  if test "x$have_plist" = "xyes"; then
    AC_DEFINE(HAVE_PLIST, 1, [Define if you have libplist])
    AC_SUBST([HEADER_HAVE_PLIST], [1])
    AC_SUBST([libplist_requires], [$LIBPLIST_REQUIRES_STR])
    AC_SUBST(libplist_CFLAGS)
    AC_SUBST(libplist_LIBS)
  else
    AC_MSG_ERROR([requested building with libplist, but library could not be found])
  fi
else
  echo "*** Note: libplist has been disabled ***"
  AC_SUBST([HEADER_HAVE_PLIST], [0])
fi

if test "x$with_openssl" == "xyes"; then
  if test "x$have_openssl" = "xyes"; then
    AC_DEFINE(HAVE_OPENSSL, 1, [Define if you have openssl])
    AC_SUBST([HEADER_HAVE_OPENSSL], [1])
    AC_SUBST([openssl_requires], [$OPENSSL_REQUIRES_STR])
    AC_SUBST(openssl_CFLAGS)
    AC_SUBST(openssl_LIBS)
    crypto_backend="openssl"
  else
    AC_MSG_ERROR([requested building with openssl, but library could not be found])
  fi
else
  echo "*** Note: openssl has been disabled ***"
  AC_SUBST([HEADER_HAVE_OPENSSL], [0])
fi

if test "x$crypto_backend" != "xnone"; then
  AC_DEFINE(HAVE_CRYPTO, 1, [Define if you have either openssl or CommCrypto])
  AC_SUBST([HEADER_HAVE_CRYPTO], [1])
else
  AC_SUBST([HEADER_HAVE_CRYPTO], [0])
fi

if test "$debug_build" = true; then
  echo "*** Note: debug build requested ***"
  CFLAGS+=" -g -O0 -DDEBUG=1"
  CXXFLAGS+=" -g -O0 -DDEBUG=1"
fi

# Checks for header files.
AC_CHECK_HEADERS([arpa/inet.h stdint.h stdlib.h string.h unistd.h winsock.h])

# Check for functions
AC_CHECK_FUNCS([memmem])



AC_CONFIG_FILES([Makefile
                 include/Makefile
                 libimg3tool.pc
                 include/img3tool/img3tool.hpp
                 img3tool/Makefile])

AC_OUTPUT


echo "
Configuration for $PACKAGE-$VERSION:
-------------------------------------------

  install prefix ..........: $prefix
  with plist ..............: $with_plist
  crypto backend ..........: $crypto_backend"

echo "  compiler ................: ${CC}

  Now type 'make' to build $PACKAGE-$VERSION,
  and then 'make install' for installation.
"
