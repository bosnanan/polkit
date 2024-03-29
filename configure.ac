dnl Process this file with autoconf to produce a configure script.

AC_PREREQ(2.59c)
AC_INIT([polkit], [0.116], [http://lists.freedesktop.org/mailman/listinfo/polkit-devel], , [http://www.freedesktop.org/wiki/Software/polkit])
AM_INIT_AUTOMAKE([])
AC_CONFIG_HEADERS(config.h)
AC_CONFIG_MACRO_DIR([buildutil])
AM_MAINTAINER_MODE

m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

# Include external mocklibc tool for unit testing
AC_CONFIG_SUBDIRS([test/mocklibc])

# libtool versioning - this applies to all libraries in this package
#
# See http://sources.redhat.com/autobook/autobook/autobook_91.html#SEC91 for details
#
LT_CURRENT=0
LT_REVISION=0
LT_AGE=0
AC_SUBST(LT_CURRENT)
AC_SUBST(LT_REVISION)
AC_SUBST(LT_AGE)

AC_USE_SYSTEM_EXTENSIONS
AC_ISC_POSIX
AC_PROG_CC
AC_HEADER_STDC
AM_PROG_LIBTOOL
AC_PROG_MAKE_SET
AC_PROG_LN_S
AC_SYS_LARGEFILE
AM_PROG_CC_C_O
AC_PROG_CXX
AX_CXX_COMPILE_STDCXX_11()

# Taken from dbus
AC_ARG_ENABLE(ansi,             [  --enable-ansi           enable -ansi -pedantic gcc flags],enable_ansi=$enableval,enable_ansi=no)
AC_ARG_ENABLE(man-pages,        [  --enable-man-pages      build manual pages],enable_man_pages=$enableval,enable_man_pages=yes)

if test "${enable_man_page}" != no; then
dnl
dnl Check for xsltproc
dnl
AC_PATH_PROG([XSLTPROC], [xsltproc])
  if test -z "$XSLTPROC"; then
    enable_man_pages=no
  fi
fi
AM_CONDITIONAL(MAN_PAGES_ENABLED, test x$enable_man_pages = xyes)

GTK_DOC_CHECK([1.3])

#### gcc warning flags

CC_CHECK_FLAGS_APPEND([WARN_CFLAGS], [CFLAGS], [\
        -pipe \
        -Wall \
	-Werror=empty-body \
	-Werror=strict-prototypes \
	-Werror=missing-prototypes \
	-Werror=implicit-function-declaration \
        "-Werror=format=2 -Werror=format-security -Werror=format-nonliteral" \
	-Werror=pointer-arith -Werror=init-self \
        -Werror=missing-declarations \
        -Werror=return-type \
        -Werror=overflow \
        -Werror=int-conversion \
        -Werror=parenthesis \
        -Werror=incompatible-pointer-types \
        -Werror=misleading-indentation \
	-Werror=missing-include-dirs -Werror=aggregate-return \
	-Werror=declaration-after-statement \
])
AC_SUBST(WARN_CFLAGS)

PKG_CHECK_MODULES(GLIB, [gmodule-2.0 gio-unix-2.0 >= 2.30.0])
AC_SUBST(GLIB_CFLAGS)
AC_SUBST(GLIB_LIBS)

PKG_CHECK_MODULES(LIBJS, [mozjs-60])

AC_SUBST(LIBJS_CFLAGS)
AC_SUBST(LIBJS_CXXFLAGS)
AC_SUBST(LIBJS_LIBS)

EXPAT_LIB=""
AC_ARG_WITH(expat, [  --with-expat=<dir>      Use expat from here],
                      [
                      expat=$withval
                      CPPFLAGS="$CPPFLAGS -I$withval/include"
                      LDFLAGS="$LDFLAGS -L$withval/lib"
                      ]
                      )
AC_CHECK_HEADERS(expat.h, [AC_DEFINE(HAVE_EXPAT_H)], 
		 [AC_MSG_ERROR([Can't find expat.h. Please install expat.])])
AC_CHECK_LIB(expat,XML_ParserCreate,[EXPAT_LIBS="-lexpat"],
	     [AC_MSG_ERROR([Can't find expat library. Please install expat.])])
AC_SUBST(EXPAT_LIBS)

AC_CHECK_FUNCS(clearenv fdatasync)

if test "x$GCC" = "xyes"; then
  LDFLAGS="-Wl,--as-needed $LDFLAGS"
fi

dnl ---------------------------------------------------------------------------
dnl - Check whether setnetgrent has a return value
dnl ---------------------------------------------------------------------------
AC_CHECK_HEADERS([netgroup.h])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
    #include <stddef.h>
    #ifdef HAVE_NETGROUP_H
    #include <netgroup.h>
    #else
    #include <netdb.h>
    #endif
]], [[int r = setnetgrent (NULL);]])],
[AC_DEFINE([HAVE_SETNETGRENT_RETURN], 1, [Define to 1 if setnetgrent has return value])])

dnl ---------------------------------------------------------------------------
dnl - Check whether we want to build test
dnl ---------------------------------------------------------------------------
AC_ARG_ENABLE([test],
              [AS_HELP_STRING([--disable-test], [Do not build tests])],
              [enable_test=$enableval], [enable_test=yes])

AM_CONDITIONAL(BUILD_TEST, [test "x$enable_test" = "xyes"])

dnl ---------------------------------------------------------------------------
dnl - Select wether to use libsystemd-login, libelogind or ConsoleKit for session tracking
dnl ---------------------------------------------------------------------------

have_libsystemd=no
have_libelogind=no
SESSION_TRACKING=ConsoleKit

AC_ARG_ENABLE([libsystemd-login],
              [AS_HELP_STRING([--enable-libsystemd-login[=@<:@auto/yes/no@:>@]], [Use libsystemd (auto/yes/no)])],
              [enable_libsystemd_login=$enableval],
              [enable_libsystemd_login=auto])
if test "$enable_libsystemd_login" != "no"; then
  PKG_CHECK_MODULES([LIBSYSTEMD],
    [libsystemd],
    [have_libsystemd=yes],
    dnl if libsystemd is not available, fall back to the older libsystemd-login
    [PKG_CHECK_MODULES([LIBSYSTEMD_LOGIN],
      [libsystemd-login],
      [
        have_libsystemd=yes
        LIBSYSTEMD_CFLAGS="$LIBSYSTEMD_LOGIN_CFLAGS"
        LIBSYSTEMD_LIBS="$LIBSYSTEMD_LOGIN_LIBS"
      ],
      [have_libsystemd=no])])
  if test "$have_libsystemd" = "yes"; then
      SESSION_TRACKING=libsystemd-login
  fi
fi

AC_ARG_ENABLE([libelogind],
              [AS_HELP_STRING([--enable-libelogind[=@<:@auto/yes/no@:>@]], [Use libelogind (auto/yes/no)])],
              [enable_libelogind=$enableval],
              [enable_libelogind=auto])
dnl Using libelogind makes no sense when libsystemd-login is already in use
if test "$have_libsystemd" = "yes"; then
  enable_libelogind=no
fi
if test "$enable_libelogind" != "no"; then
  PKG_CHECK_MODULES([LIBELOGIND],
    [libelogind],
    [have_libelogind=yes])
  if test "$have_libelogind" = "yes"; then
      SESSION_TRACKING=libelogind
  fi
fi

dnl libelogind is a drop-in replacement for libsystemd-login, so using it does
dnl not need any different actions than using libsystemd-login.
if test "$SESSION_TRACKING" != "ConsoleKit"; then
  AC_DEFINE([HAVE_LIBSYSTEMD], 1, [Define to 1 if libsystemd or libelogind is available])
  save_LIBS=$LIBS
  if test "$have_libelogind" = "yes"; then
    LIBS=$LIBELOGIND_LIBS
    dnl We have to act like this was libsystemd-login
    LIBSYSTEMD_CFLAGS=$LIBELOGIND_CFLAGS
    LIBSYSTEMD_LIBS=$LIBELOGIND_LIBS
  else
    LIBS=$LIBSYSTEMD_LIBS
  fi
  AC_CHECK_FUNCS(sd_uid_get_display)
  LIBS=$save_LIBS
else
  if test "$enable_libsystemd_login" = "yes"; then
    AC_MSG_ERROR([libsystemd support requested but libsystemd or libsystemd-login library not found])
  fi
  if test "$enable_libelogind" = "yes"; then
    AC_MSG_ERROR([libelogind support requested but libelogind library not found])
  fi
fi

AS_IF([test "x$cross_compiling" != "xyes" ], [
  AS_IF([test "$have_libsystemd" = "yes"], [
    AS_IF([test ! -d /sys/fs/cgroup/systemd/ -a ! -d /sys/fs/cgroup/init.scope ], [
      AS_IF([test "$enable_libsystemd_login" = "yes"], [
        AC_MSG_WARN([libsystemd requested but system does not appear to be using systemd])
      ], [
        AC_MSG_ERROR([libsystemd autoconfigured, but system does not appear to use systemd])
      ])
    ])
  ], [
    AS_IF([test -d /sys/fs/cgroup/systemd/ ], [
      AS_IF([test "$enable_libsystemd_login" = "no" ], [
        AC_MSG_WARN([ConsoleKit requested but system appears to use systemd])
      ], [
        AC_MSG_ERROR([ConsoleKit autoconfigured, but systemd is in use (missing libsystemd or libsystemd-login pkg-config?)])
      ])
    ])
  ])
])

AC_SUBST(LIBSYSTEMD_CFLAGS)
AC_SUBST(LIBSYSTEMD_LIBS)
AM_CONDITIONAL(HAVE_LIBSYSTEMD, [test "$have_libsystemd" = "yes"], [Using libsystemd])

dnl ---------------------------------------------------------------------------
dnl - systemd unit / service files
dnl ---------------------------------------------------------------------------

AC_ARG_WITH([systemdsystemunitdir],
            AS_HELP_STRING([--with-systemdsystemunitdir=DIR], [Directory for systemd service files]),
            [],
            [with_systemdsystemunitdir=$($PKG_CONFIG --variable=systemdsystemunitdir systemd)])
if test "x$with_systemdsystemunitdir" != "xno"; then
  AC_SUBST([systemdsystemunitdir], [$with_systemdsystemunitdir])
fi
AM_CONDITIONAL(HAVE_SYSTEMD, [test -n "$systemdsystemunitdir"])

dnl ---------------------------------------------------------------------------
dnl - User for running polkitd
dnl ---------------------------------------------------------------------------

AC_ARG_WITH(polkitd_user, AS_HELP_STRING([--with-polkitd-user=<user>],[User for running polkitd (polkitd)]))

if test -z "$with_polkitd_user" ; then
    POLKITD_USER=polkitd
else
    POLKITD_USER=$with_polkitd_user
fi
AC_SUBST(POLKITD_USER)
AC_DEFINE_UNQUOTED(POLKITD_USER,"$POLKITD_USER", [User for running polkitd])

dnl ---------------------------------------------------------------------------
dnl - Select which authentication framework to use
dnl ---------------------------------------------------------------------------

AC_ARG_WITH([authfw],
	    AS_HELP_STRING([--with-authfw=<name>], 
                           [Authentication framework (none/pam/shadow)]))
if ! test -z "$with_authfw" ; then
  if test x$with_authdb = xdummy ; then
    if ! test x$with_authfw = xnone ; then
      AC_MSG_ERROR([Only 'none' is a valid authentication framework for the dummy authorization database])
    fi
  else
      if test x$with_authfw = xnone ; then
        AC_MSG_ERROR(['none' is only a valid authentication framework for the dummy authorization database])
      fi
  fi
  POLKIT_AUTHFW=$with_authfw
else
  if test x$with_authdb = xdummy ; then
    POLKIT_AUTHFW=none
  else
    POLKIT_AUTHFW=pam
  fi
fi

AC_SUBST(POLKIT_AUTHFW)
AC_DEFINE_UNQUOTED(POLKIT_AUTHFW,"$POLKIT_AUTHFW", [Authentication Framework to use])

case $POLKIT_AUTHFW in
  none)
    need_pam=no
    AC_DEFINE(POLKIT_AUTHFW_NONE, 1, [If using no authentication framework])
    ;;

  pam)
    need_pam=yes
    AC_DEFINE(POLKIT_AUTHFW_PAM, 1, [If using the PAM authentication framework])
    ;;

  shadow)
    need_pam=no
    AUTH_LIBS="${AUTH_LIBS} -lcrypt"
    AC_DEFINE(POLKIT_AUTHFW_SHADOW, 1, [If using the Shadow authentication framework])
    ;;

  bsdauth)
    need_pam=no
    AC_DEFINE(POLKIT_AUTHFW_BSDAUTH, 1, [If using the bsd_auth(3) authentication framework])
    ;;

  *)
    AC_MSG_ERROR([Unknown Authentication Framework: $POLKIT_AUTHFW])
    ;;
esac

AM_CONDITIONAL(POLKIT_AUTHFW_NONE, [test x$POLKIT_AUTHFW = xnone], [Using no authfw])
AM_CONDITIONAL(POLKIT_AUTHFW_PAM, [test x$POLKIT_AUTHFW = xpam], [Using PAM authfw])
AM_CONDITIONAL(POLKIT_AUTHFW_SHADOW, [test x$POLKIT_AUTHFW = xshadow], [Using Shadow authfw])
AM_CONDITIONAL(POLKIT_AUTHFW_BSDAUTH, [test x$POLKIT_AUTHFW = xbsdauth], [Using bsd_auth(3) authfw])


dnl ---------------------------------------------------------------------------
dnl - Check for PAM
dnl ---------------------------------------------------------------------------

withval=""
AC_ARG_WITH(pam-prefix,
[  --with-pam-prefix=<prefix> specify where pam files go],[
if test x$withval != x; then
   AC_MSG_RESULT("PAM files will be installed in prefix ${withval}.")
fi])
if test x$withval != x; then
	PAM_PREFIX_UNEXPANDED="$withval"
else
	PAM_PREFIX_UNEXPANDED="$sysconfdir"
fi
PAM_PREFIX=`eval echo $PAM_PREFIX_UNEXPANDED`
AC_SUBST(PAM_PREFIX)

have_pam=no
if test "$need_pam" = yes ; then
  AC_CHECK_LIB(pam, pam_start, have_pam=yes)
fi

if test x$have_pam = xno; then
    if test "$need_pam" = yes ; then
      AC_ERROR([Could not find pam/pam-devel, please install the needed packages.])
    fi
else
    AUTH_LIBS="${AUTH_LIBS} -lpam"
    AC_DEFINE(HAVE_PAM, 1, [Define if PAM support is included])

    # On Linux, sigtimedwait() is in libc; on Solaris, it's in librt.
    have_timedwait=no
    AC_CHECK_LIB(c, sigtimedwait, [have_timedwait=yes])
    if test "$have_timedwait" = no ; then
      AC_CHECK_LIB(rt, sigtimedwait, [AUTH_LIBS="${AUTH_LIBS} -lrt"])
    fi

    AC_MSG_CHECKING(how to call pam_strerror)
    AC_CACHE_VAL(ac_cv_pam_strerror_args,
     [AC_TRY_COMPILE([#include <stdio.h>
                      #include <stdlib.h>
                      #include <security/pam_appl.h>],
                     [pam_handle_t *pamh = 0;
                      char *s = pam_strerror(pamh, PAM_SUCCESS);],
                     [ac_pam_strerror_args=2],
                     [AC_TRY_COMPILE([#include <stdio.h>
                                      #include <stdlib.h>
                                      #include <security/pam_appl.h>],
                                     [char *s =
                                       pam_strerror(PAM_SUCCESS);],
                                     [ac_pam_strerror_args=1],
                                     [ac_pam_strerror_args=0])])
      ac_cv_pam_strerror_args=$ac_pam_strerror_args])
    ac_pam_strerror_args=$ac_cv_pam_strerror_args
    if test "$ac_pam_strerror_args" = 1 ; then
      AC_MSG_RESULT(one argument)
    elif test "$ac_pam_strerror_args" = 2 ; then
      AC_DEFINE(PAM_STRERROR_TWO_ARGS, 1, [Define if pam_strerror takes two arguments])
      AC_MSG_RESULT(two arguments)
    else
      AC_MSG_RESULT(unknown)
    fi
 
fi

AM_CONDITIONAL(HAVE_PAM, test x$have_pam = xyes)
AC_SUBST(HAVE_PAM)
AC_SUBST(AUTH_LIBS)

AC_CHECK_HEADER(security/pam_modutil.h, [AC_DEFINE(HAVE_PAM_MODUTIL_H, [], "Have pam_modutil.h")])
AC_CHECK_HEADER(security/pam_ext.h, [AC_DEFINE(HAVE_PAM_EXT_H, [], "Have pam_ext.h")])
AC_CHECK_LIB(pam, pam_vsyslog, [AC_DEFINE(HAVE_PAM_VSYSLOG, [], "Have pam_vsyslog")])

AC_ARG_WITH(pam-module-dir, [  --with-pam-module-dir=[dirname]  directory to install PAM security module])
if ! test -z "$with_pam_module_dir"; then
    PAM_MODULE_DIR=$with_pam_module_dir
else
    PAM_MODULE_DIR="/lib/security"
fi

AC_SUBST(PAM_MODULE_DIR)

AC_ARG_WITH(os-type,     [  --with-os-type=<os>        distribution or OS (redhat/suse/gentoo/pardus/solaris/netbsd)])

#### Check our operating system (distro-tweaks required)
if test "z$with_os_type" = "z"; then
	AC_CHECK_FILE(/etc/redhat-release,distro_type="redhat")
	AC_CHECK_FILE(/etc/SuSE-release,distro_type="suse")
	AC_CHECK_FILE(/etc/gentoo-release,distro_type="gentoo")
	AC_CHECK_FILE(/etc/pardus-release,distro_type="pardus")
	if test "z$distro_type" = "z"; then
		echo "Linux distribution autodetection failed, specify the distribution to target using --with-os-type="
	else
		operating_system=`echo ${distro_type} | tr '[[:upper:]]' '[[:lower:]]' `
	fi
fi

#### Sort out OS (distro-tweaks required)
if test x$with_os_type = x; then
    if test x$operating_system = xredhat ; then
        with_os_type=redhat
    elif test x$operating_system = xsuse ; then
        with_os_type=suse
    elif test x$operating_system = xgentoo ; then
        with_os_type=gentoo
    elif test x$operating_system = xpardus ; then
        with_os_type=pardus
    elif test x$operating_system = xsolaris ; then
	with_os_type=solaris
    elif test x$operating_system = xfreebsd ; then
	with_os_type=freebsd
    elif test x$operating_system = xnetbsd ; then
	with_os_type=netbsd
    else
	case "$host_os" in
	  *netbsd*)
		with_os_type=netbsd
		;;
	  *)
		with_os_type=unknown
		::
	esac
    fi
fi

# (distro-tweaks required)
AM_CONDITIONAL(OS_TYPE_UNKNOWN, test x$with_os_type = xunknown, [Running on unknown OS])
AM_CONDITIONAL(OS_TYPE_RED_HAT, test x$with_os_type = xredhat, [Running on Red Hat OS'es])
AM_CONDITIONAL(OS_TYPE_SUSE, test x$with_os_type = xsuse, [Running on SUSE OS'es])
AM_CONDITIONAL(OS_TYPE_GENTOO, test x$with_os_type = xgentoo, [Running on Gentoo OS'es])
AM_CONDITIONAL(OS_TYPE_PARDUS, test x$with_os_type = xpardus, [Running on Pardus OS'es])
AM_CONDITIONAL(OS_TYPE_SOLARIS, test x$with_os_type = xsolaris, [Running os Solaris OS'es])
AM_CONDITIONAL(OS_TYPE_FREEBSD, test x$with_os_type = xfreebsd, [Running on FreeBSD OS'es])

AC_ARG_WITH(pam-include, [  --with-pam-include=<file>  pam file to include])

#### Set up pam file to include (distro-tweaks required)
if ! test -z "$with_pam_include"; then
   PAM_FILE_INCLUDE_AUTH=$with_pam_include
   PAM_FILE_INCLUDE_ACCOUNT=$with_pam_include
   PAM_FILE_INCLUDE_PASSWORD=$with_pam_include
   PAM_FILE_INCLUDE_SESSION=$with_pam_include
elif test x$with_os_type = xredhat -o x$with_os_type = xgentoo -o x$with_os_type = xpardus ; then
   PAM_FILE_INCLUDE_AUTH=system-auth
   PAM_FILE_INCLUDE_ACCOUNT=system-auth
   PAM_FILE_INCLUDE_PASSWORD=system-auth
   PAM_FILE_INCLUDE_SESSION=system-auth
elif test x$with_os_type = xsuse -o x$with_os_type = xsolaris ; then
   PAM_FILE_INCLUDE_AUTH=common-auth
   PAM_FILE_INCLUDE_ACCOUNT=common-account
   PAM_FILE_INCLUDE_PASSWORD=common-password
   PAM_FILE_INCLUDE_SESSION=common-session
elif test x$with_os_type = xfreebsd -o x$with_os_type = xnetbsd; then
    PAM_FILE_INCLUDE_AUTH=system
    PAM_FILE_INCLUDE_ACCOUNT=system
    PAM_FILE_INCLUDE_PASSWORD=system
    PAM_FILE_INCLUDE_SESSION=system
else
   PAM_FILE_INCLUDE_AUTH=system-auth
   PAM_FILE_INCLUDE_ACCOUNT=system-auth
   PAM_FILE_INCLUDE_PASSWORD=system-auth
   PAM_FILE_INCLUDE_SESSION=system-auth
fi

AC_SUBST(PAM_FILE_INCLUDE_AUTH)
AC_SUBST(PAM_FILE_INCLUDE_ACCOUNT)
AC_SUBST(PAM_FILE_INCLUDE_PASSWORD)
AC_SUBST(PAM_FILE_INCLUDE_SESSION)
AC_DEFINE_UNQUOTED(PAM_FILE_INCLUDE_AUTH, "$PAM_FILE_INCLUDE_AUTH", [pam file auth])
AC_DEFINE_UNQUOTED(PAM_FILE_INCLUDE_ACCOUNT, "$PAM_FILE_INCLUDE_ACCOUNT", [pam file account])
AC_DEFINE_UNQUOTED(PAM_FILE_INCLUDE_PASSWORD, "$PAM_FILE_INCLUDE_PASSWORD", [pam file password])
AC_DEFINE_UNQUOTED(PAM_FILE_INCLUDE_SESSION, "$PAM_FILE_INCLUDE_SESSION", [pam file session])

dnl ---------------------------------------------------------------------------
dnl - check OS
dnl ---------------------------------------------------------------------------
case "$host_os" in
  *linux*)
	;;
  *solaris*)
	AC_DEFINE([HAVE_SOLARIS], 1, [Is this a Solaris system?])
	;;
  *freebsd*)
  	AC_DEFINE([HAVE_FREEBSD], 1, [Is this a FreeBSD system?])
	;;
  *netbsd*)
	AC_DEFINE([HAVE_NETBSD], 1, [Is this an NetBSD system?])
	;;
  *openbsd*)
	AC_DEFINE([HAVE_OPENBSD], 1, [Is this an OpenBSD system?])
	;;
esac

GOBJECT_INTROSPECTION_CHECK([0.6.2])

AC_ARG_ENABLE([examples],
              AS_HELP_STRING([--enable-examples], [Build the example programs]),,
              [enable_examples=yes])

AM_CONDITIONAL(BUILD_EXAMPLES, test "x$enable_examples" = "xyes")

# ********************
# Internationalization
# ********************

IT_PROG_INTLTOOL([0.40.0])
GETTEXT_PACKAGE=polkit-1
AC_SUBST([GETTEXT_PACKAGE])
AM_GLIB_GNU_GETTEXT
AC_DEFINE_UNQUOTED([GETTEXT_PACKAGE],["$GETTEXT_PACKAGE"],[gettext domain])

AC_ARG_VAR([SUID_CFLAGS],
           [CFLAGS used for binaries which are usually with the suid bit])
AC_ARG_VAR([SUID_LDFLAGS],
           [LDFLAGS used for binaries which are usually with the suid bit])

AC_OUTPUT([
Makefile
actions/Makefile
data/Makefile
data/polkit-1
data/polkit-gobject-1.pc
data/polkit-agent-1.pc
src/Makefile
src/polkit/Makefile
src/polkitbackend/Makefile
src/polkitagent/Makefile
src/programs/Makefile
src/examples/Makefile
docs/version.xml
docs/extensiondir.xml
docs/Makefile
docs/polkit/Makefile
docs/man/Makefile
po/Makefile.in
test/Makefile
test/polkit/Makefile
test/polkitbackend/Makefile
])

dnl ==========================================================================
echo "
                  polkit $VERSION
                =================

        prefix:                     ${prefix}
        libdir:                     ${libdir}
        libexecdir:                 ${libexecdir}
        bindir:                     ${bindir}
        sbindir:                    ${sbindir}
        datadir:                    ${datadir}
        sysconfdir:                 ${sysconfdir}
        localstatedir:              ${localstatedir}
        docdir:                     ${docdir}

        compiler:                   ${CC}
        cflags:                     ${CFLAGS}
        cppflags:                   ${CPPFLAGS}
        xsltproc:                   ${XSLTPROC}
	introspection:		    ${found_introspection}

        Distribution/OS:            ${with_os_type}
        Authentication framework:   ${POLKIT_AUTHFW}
        Session tracking:           ${SESSION_TRACKING}
        PAM support:                ${have_pam}
        systemdsystemunitdir:       ${systemdsystemunitdir}
        polkitd user:               ${POLKITD_USER}"

if test "$have_pam" = yes ; then
echo "
        PAM file auth:              ${PAM_FILE_INCLUDE_AUTH}
        PAM file account:           ${PAM_FILE_INCLUDE_ACCOUNT}
        PAM file password:          ${PAM_FILE_INCLUDE_PASSWORD}
        PAM file session:           ${PAM_FILE_INCLUDE_SESSION}"
fi
echo "
        Maintainer mode:            ${USE_MAINTAINER_MODE}
        Building api docs:          ${enable_gtk_doc}
        Building man pages:         ${enable_man_pages}
        Building examples:          ${enable_examples}

"

echo "NOTE: The file ${prefix}/lib/polkit-1/polkit-agent-helper-1 must be owned"
echo "      by root and have mode 4755 (setuid root binary)"
echo

echo "NOTE: The file ${bindir}/pkexec must be owned by root and"
echo "      have mode 4755 (setuid root binary)"
echo

echo "NOTE: The directory ${sysconfdir}/polkit-1/rules.d must be owned"
echo "      by user '$POLKITD_USER' and have mode 700"
echo

echo "NOTE: The directory ${datadir}/polkit-1/rules.d must be owned"
echo "      by user '$POLKITD_USER' and have mode 700"
echo
