AC_PREREQ([2.64])
AC_INIT([weston],
        [0.95.0],
        [https://bugs.freedesktop.org/enter_bug.cgi?product=weston],
        [weston],
        [http://wayland.freedesktop.org/])

AC_CONFIG_HEADERS([config.h])

AM_INIT_AUTOMAKE([1.11 foreign no-dist-gzip dist-xz])

AM_SILENT_RULES([yes])

# Check for programs
AC_PROG_CC
AC_PROG_CXX

# Initialize libtool
LT_PREREQ([2.2])
LT_INIT([disable-static])

PKG_PROG_PKG_CONFIG()

AC_CHECK_FUNC([dlopen], [],
              AC_CHECK_LIB([dl], [dlopen], DLOPEN_LIBS="-ldl"))
AC_SUBST(DLOPEN_LIBS)

AC_CHECK_HEADERS([execinfo.h])

AC_CHECK_FUNCS([mkostemp strchrnul])

PKG_CHECK_MODULES(COMPOSITOR,
		  [wayland-server egl >= 7.10 glesv2 xkbcommon pixman-1])


AC_ARG_ENABLE(setuid-install, [  --enable-setuid-install],,
	      enable_setuid_install=yes)
AM_CONDITIONAL(ENABLE_SETUID_INSTALL, test x$enable_setuid_install = xyes)


AC_ARG_ENABLE(xwayland, [  --enable-xwayland],,
	      enable_xwayland=yes)
AM_CONDITIONAL(ENABLE_XWAYLAND, test x$enable_xwayland = xyes)
if test x$enable_xwayland = xyes; then
  PKG_CHECK_MODULES([XWAYLAND], xcb xcb-xfixes xcursor cairo-xcb)
  AC_DEFINE([BUILD_XWAYLAND], [1], [Build the X server launcher])

  AC_ARG_WITH(xserver-path, AS_HELP_STRING([--with-xserver-path=PATH],
              [Path to X server]), [XSERVER_PATH="$withval"],
              [XSERVER_PATH="$bindir/Xorg"])
  AC_SUBST([XSERVER_PATH])
fi


AC_ARG_ENABLE(x11-compositor, [  --enable-x11-compositor],,
	      enable_x11_compositor=yes)
AM_CONDITIONAL(ENABLE_X11_COMPOSITOR, test x$enable_x11_compositor = xyes)
if test x$enable_x11_compositor = xyes; then
  PKG_CHECK_MODULES([XCB], xcb)
  xcb_save_LIBS=$LIBS
  xcb_save_CFLAGS=$CFLAGS
  CFLAGS=$XCB_CFLAGS
  LIBS=$XCB_LIBS
  AC_CHECK_FUNCS([xcb_poll_for_queued_event])
  LIBS=$xcb_save_LIBS
  CFLAGS=$xcb_save_CFLAGS

  X11_COMPOSITOR_MODULES="x11 x11-xcb"

  PKG_CHECK_MODULES(X11_COMPOSITOR_XKB, [xcb-xkb],
		    [have_xcb_xkb="yes"], [have_xcb_xkb="no"])
  if test "x$have_xcb_xkb" = xyes; then
	# Most versions of XCB have totally broken XKB bindings, where the
	# events don't work.  Make sure we can actually use them.
	xcb_xkb_save_CFLAGS=$CFLAGS
	CFLAGS=$X11_COMPOSITOR_XKB_CFLAGS
	AC_CHECK_MEMBER([struct xcb_xkb_state_notify_event_t.xkbType],
			[], [have_xcb_xkb=no], [[#include <xcb/xkb.h>]])
	CFLAGS=$xcb_xkb_save_CFLAGS
  fi
  if test "x$have_xcb_xkb" = xyes; then
	X11_COMPOSITOR_MODULES="$X11_COMPOSITOR_MODULES xcb-xkb"
	AC_DEFINE([HAVE_XCB_XKB], [1], [libxcb supports XKB protocol])
  fi

  PKG_CHECK_MODULES(X11_COMPOSITOR, [$X11_COMPOSITOR_MODULES])
  AC_DEFINE([BUILD_X11_COMPOSITOR], [1], [Build the X11 compositor])
fi


AC_ARG_ENABLE(drm-compositor, [  --enable-drm-compositor],,
	      enable_drm_compositor=yes)
AM_CONDITIONAL(ENABLE_DRM_COMPOSITOR, test x$enable_drm_compositor = xyes)
if test x$enable_drm_compositor = xyes; then
  AC_DEFINE([BUILD_DRM_COMPOSITOR], [1], [Build the DRM compositor])
  PKG_CHECK_MODULES(DRM_COMPOSITOR, [libudev >= 136 libdrm >= 2.4.30 gbm mtdev >= 1.1.0])
fi


AC_ARG_ENABLE(wayland-compositor, [  --enable-wayland-compositor],,
	      enable_wayland_compositor=yes)
AM_CONDITIONAL(ENABLE_WAYLAND_COMPOSITOR,
	       test x$enable_wayland_compositor = xyes)
if test x$enable_wayland_compositor = xyes; then
  AC_DEFINE([BUILD_WAYLAND_COMPOSITOR], [1],
	    [Build the Wayland (nested) compositor])
  PKG_CHECK_MODULES(WAYLAND_COMPOSITOR, [wayland-client wayland-egl])
fi


AC_ARG_ENABLE(android-compositor,
	      AS_HELP_STRING([--disable-android-compositor],
	                     [do not build-test the Android 4.0 backend]),,
	      enable_android_compositor=yes)
AM_CONDITIONAL(ENABLE_ANDROID_COMPOSITOR, test x$enable_android_compositor = xyes)
if test x$enable_android_compositor = xyes; then
  AC_DEFINE([BUILD_ANDROID_COMPOSITOR], [1], [Build the compositor for Android 4.0])
  PKG_CHECK_MODULES(ANDROID_COMPOSITOR, [mtdev >= 1.1.0])
fi


AC_ARG_WITH(cairo-glesv2, AS_HELP_STRING([--with-cairo-glesv2],
              [Use GLESv2 cairo instead of full GL]),
              [cairo_modules="cairo-glesv2"],
              [cairo_modules="cairo-gl"])
if test x$cairo_modules = xcairo-glesv2; then
  AC_DEFINE([USE_CAIRO_GLESV2], [1], [Use the GLESv2 GL cairo backend])
fi

PKG_CHECK_MODULES(PIXMAN, [pixman-1])
PKG_CHECK_MODULES(PNG, [libpng])
PKG_CHECK_MODULES(WEBP, [libwebp], [have_webp=yes], [have_webp=no])
AS_IF([test "x$have_webp" = "xyes"],
      [AC_DEFINE([HAVE_WEBP], [1], [Have webp])])

AC_CHECK_LIB([jpeg], [jpeg_CreateDecompress], have_jpeglib=yes)
if test x$have_jpeglib = xyes; then
  IMAGE_LIBS="$IMAGE_LIBS -ljpeg"
else
  AC_ERROR([libjpeg not found])
fi

PKG_CHECK_MODULES(CAIRO, [cairo])
SHARED_LIBS="$CAIRO_LIBS $PIXMAN_LIBS $PNG_LIBS $WEBP_LIBS $IMAGE_LIBS"
SHARED_CFLAGS="$CAIRO_CFLAGS $PIXMAN_CFLAGS $PNG_CFLAGS $WEBP_CFLAGS"
AC_SUBST(SHARED_LIBS)
AC_SUBST(SHARED_CFLAGS)

COMPOSITOR_LIBS="$COMPOSITOR_LIBS $IMAGE_LIBS"
COMPOSITOR_CFLAGS="$COMPOSITOR_CFLAGS $IMAGE_CFLAGS"

AC_ARG_ENABLE(simple-clients, [  --enable-simple-clients],, enable_simple_clients=yes)
AM_CONDITIONAL(BUILD_SIMPLE_CLIENTS, test x$enable_simple_clients = xyes)
if test x$enable_simple_clients = xyes; then
  AC_DEFINE([BUILD_SIMPLE_CLIENTS], [1], [Build the Wayland simple clients])
  PKG_CHECK_MODULES(SIMPLE_CLIENT,
                    [egl >= 7.10 glesv2 wayland-client wayland-egl])
fi

AC_ARG_ENABLE(clients, [  --enable-clients],, enable_clients=yes)
AM_CONDITIONAL(BUILD_CLIENTS, test x$enable_clients = xyes)
if test x$enable_clients = xyes; then
  AC_DEFINE([BUILD_CLIENTS], [1], [Build the Wayland clients])

  PKG_CHECK_MODULES(CLIENT, [wayland-client wayland-egl egl >= 7.10 cairo >= 1.10.0 xkbcommon wayland-cursor])
  PKG_CHECK_MODULES(WESTON_INFO, [wayland-client])

  CLIENT_CFLAGS="$CLIENT_CFLAGS $IMAGE_CFLAGS"
  CLIENT_LIBS="$CLIENT_LIBS $IMAGE_LIBS"

  PKG_CHECK_MODULES(POPPLER, [poppler-glib glib-2.0 gobject-2.0 gio-2.0 ],
			     [have_poppler=yes], [have_poppler=no])
  PKG_CHECK_MODULES(CAIRO_EGL, [cairo-egl >= 1.11.3 $cairo_modules],
		    [have_cairo_egl=yes], [have_cairo_egl=no])
  AS_IF([test "x$have_cairo_egl" = "xyes"],
        [AC_DEFINE([HAVE_CAIRO_EGL], [1], [Have cairo-egl])],
        [AC_MSG_WARN([Cairo-EGL not found - clients will use cairo image])])
fi

AC_ARG_ENABLE(weston-launch, [  --enable-weston-launch],, enable_weston_launch=yes)
AM_CONDITIONAL(BUILD_WESTON_LAUNCH, test x$enable_weston_launch == xyes)
if test x$enable_weston_launch == xyes; then
  PKG_CHECK_MODULES(WESTON_LAUNCH, [libdrm])
  PKG_CHECK_MODULES(SYSTEMD_LOGIN, [libsystemd-login],
		    [have_systemd_login=yes], [have_systemd_login=no])
  AS_IF([test "x$have_systemd_login" = "xyes"],
	[AC_DEFINE([HAVE_SYSTEMD_LOGIN], [1], [Have systemd-login])])

  AC_CHECK_LIB([pam], [pam_open_session], [have_pam=yes], [have_pam=no])
  if test x$have_pam == xno; then
    AC_ERROR([weston-launch requires pam])
  fi
  WESTON_LAUNCH_LIBS="$WESTON_LAUNCH_LIBS -lpam"
fi

AM_CONDITIONAL(HAVE_POPPLER, test "x$have_poppler" = "xyes")

AM_CONDITIONAL(BUILD_FULL_GL_CLIENTS,
	       test x$cairo_modules = "xcairo-gl" -a "x$have_cairo_egl" = "xyes")

AM_CONDITIONAL(ENABLE_DESKTOP_SHELL, true)

AC_ARG_ENABLE(tablet-shell, [  --enable-tablet-shell],,
	      enable_tablet_shell=yes)
AM_CONDITIONAL(ENABLE_TABLET_SHELL,
	       test x$enable_tablet_shell = xyes)


AC_ARG_ENABLE(wcap-tools, [  --disable-wcap-tools],, enable_wcap_tools=yes)
AM_CONDITIONAL(BUILD_WCAP_TOOLS, test x$enable_wcap_tools = xyes)
if test x$enable_wcap_tools = xyes; then
  AC_DEFINE([BUILD_WCAP_TOOLS], [1], [Build the wcap tools])
  PKG_CHECK_MODULES(WCAP, [cairo])
  WCAP_LIBS="$WCAP_LIBS -lm"
fi

AC_CHECK_PROG(RSVG_CONVERT, rsvg-convert, rsvg-convert)
AM_CONDITIONAL(HAVE_RSVG_CONVERT, test -n "$RSVG_CONVERT")

PKG_CHECK_MODULES(SETBACKLIGHT, [libudev libdrm], enable_setbacklight=yes, enable_setbacklight=no)
AM_CONDITIONAL(BUILD_SETBACKLIGHT, test "x$enable_setbacklight" = "xyes")

if test "x$GCC" = "xyes"; then
	my_common_gcc_flags="-Wall -Wextra -Wno-unused-parameter \
		-Wno-missing-field-initializers -g -fvisibility=hidden"
	GCC_CFLAGS="$my_common_gcc_flags \
		-Wstrict-prototypes -Wmissing-prototypes"
	GCC_CXXFLAGS="$my_common_gcc_flags"
fi
AC_SUBST(GCC_CFLAGS)
AC_SUBST(GCC_CXXFLAGS)

WAYLAND_SCANNER_RULES(['$(top_srcdir)/protocol'])

AC_CONFIG_FILES([Makefile
		 shared/Makefile
		 src/Makefile
		 src/xwayland/Makefile
		 clients/Makefile
		 wcap/Makefile
		 data/Makefile
		 protocol/Makefile
		 tests/Makefile])
AC_OUTPUT
