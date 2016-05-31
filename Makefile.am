# {{{ global flags

BUILT_SOURCES =
CLEANFILES =
EXTRA_DIST =
DISTCLEANFILES =

AM_CFLAGS = \
	$(GLIB_CFLAGS) \
	$(OPENSSL_CFLAGS) \
	$(LIBCURL_CFLAGS) \
	-I$(srcdir)/libtools \
	-I$(srcdir)/mega \
	-I$(srcdir)

# }}}
# {{{ libmega

lib_LTLIBRARIES = \
	libmega.la

libmega_la_CFLAGS = \
	$(AM_CFLAGS) \
	-DG_LOG_DOMAIN=\"Mega\"

libmega_la_LIBADD = \
	$(GLIB_LIBS) \
	$(OPENSSL_LIBS) \
	$(LIBCURL_LIBS)

libmega_la_LDFLAGS = -version-info $(LIB_MEGA_VERSION) -no-undefined

libmega_la_SOURCES = \
	mega/mega-aes-key.c \
	mega/mega-aes-key.h \
	mega/mega-rsa-key.c \
	mega/mega-rsa-key.h \
	mega/mega-chunked-cbc-mac.c \
	mega/mega-chunked-cbc-mac.h \
	mega/mega-aes-ctr-encryptor.c \
	mega/mega-aes-ctr-encryptor.h \
	mega/utils.c \
	mega/utils.h \
	mega/mega.h \
	$(GLIB_GENERATED)

include $(GLIB_MAKEFILE)

GLIB_GENERATED = \
	mega/mega-marshal.c \
	mega/mega-marshal.h \
	mega/mega-enum-types.c \
	mega/mega-enum-types.h

mega_marshal_sources = $(libmega_la_SOURCES)
mega_enum_types_sources = $(libmega_la_SOURCES)

BUILT_SOURCES += $(GLIB_GENERATED)

pkginclude_HEADERS = $(filter-out mega/mega-marshal.h mega/private-utils.h,$(filter %.h,$(libmega_la_SOURCES)))
pkgincludedir = $(includedir)/mega

include $(INTROSPECTION_MAKEFILE)
INTROSPECTION_GIRS = Mega-1.0.gir
INTROSPECTION_SCANNER_ARGS = --add-include-path=.
INTROSPECTION_COMPILER_ARGS = --includedir=.

if HAVE_INTROSPECTION

gi_libmega_files = $(filter-out mega/mega-enum-types.% mega/mega-marshal.%,$(libmega_la_SOURCES))
gi_built_libmega_files = mega/mega-enum-types.h

Mega-1.0.gir: libmega.la
Mega_1_0_gir_INCLUDES = Gio-2.0
Mega_1_0_gir_CFLAGS = $(AM_CFLAGS)
Mega_1_0_gir_LIBS = libmega.la
Mega_1_0_gir_EXPORT_PACKAGES = libmega
Mega_1_0_gir_SCANNERFLAGS = --c-include "mega/mega.h" --warn-all
Mega_1_0_gir_FILES = \
  $(addprefix $(srcdir)/, $(gi_libmega_files)) \
  $(foreach f,$(gi_built_libmega_files), \
     $(if $(shell test -f $(addprefix $(srcdir)/,$(f)) && echo yes), \
	$(addprefix $(srcdir)/,$(f)), \
	$(f)))

girdir = $(datadir)/gir-1.0
gir_DATA = $(INTROSPECTION_GIRS)

typelibdir = $(libdir)/girepository-1.0
typelib_DATA = $(INTROSPECTION_GIRS:.gir=.typelib)

CLEANFILES += $(gir_DATA) $(typelib_DATA)

endif

# }}}
# {{{ tools

EXTRA_DIST += \
	libtools/sjson.c

TOOLS_SOURCES = \
	libtools/sjson.gen.c \
	libtools/sjson.h \
	libtools/http.c \
	libtools/http.h \
	libtools/oldmega.c \
	libtools/oldmega.h \
	libtools/tools.c \
	libtools/tools.h \
	libtools/alloc.h

LDADD = \
	libmega.la \
	$(GLIB_LIBS) \
	$(OPENSSL_LIBS) \
	$(LIBCURL_LIBS)

megafs_LDADD = \
	$(LDADD) \
	$(FUSE_LIBS)

megafs_CFLAGS = \
	$(AM_CFLAGS) \
	$(FUSE_CFLAGS)

bin_PROGRAMS = megadf megadl megaget megals megamkdir megaput megareg megarm megacopy

megadf_SOURCES     = tools/df.c $(TOOLS_SOURCES)
megadl_SOURCES     = tools/dl.c $(TOOLS_SOURCES)
megaget_SOURCES    = tools/get.c $(TOOLS_SOURCES)
megals_SOURCES     = tools/ls.c $(TOOLS_SOURCES)
megamkdir_SOURCES  = tools/mkdir.c $(TOOLS_SOURCES)
megaput_SOURCES    = tools/put.c $(TOOLS_SOURCES)
megareg_SOURCES    = tools/reg.c $(TOOLS_SOURCES)
megarm_SOURCES     = tools/rm.c $(TOOLS_SOURCES)
megacopy_SOURCES   = tools/copy.c $(TOOLS_SOURCES)

if ENABLE_FUSE
bin_PROGRAMS += megafs
megafs_SOURCES     = tools/fs.c $(TOOLS_SOURCES)
endif

# }}}
# {{{ docs

MAN1 = megadf megadl megaget megals megamkdir megaput megareg megarm megacopy megafs
MAN5 = megarc
MAN7 = megatools

man1_MANS = $(addprefix docs/,$(addsuffix .1, $(MAN1)))
man5_MANS = $(addprefix docs/,$(addsuffix .5, $(MAN5)))
man7_MANS = $(addprefix docs/,$(addsuffix .7, $(MAN7)))

MANS_ALL = $(man1_MANS) $(man5_MANS) $(man7_MANS)

BUILT_SOURCES += $(MANS_ALL)

EXTRA_DIST += \
	docs/asciidoc.conf \
	docs/footer.txt \
	docs/remote-paths.txt \
	docs/shared-options.txt \
	$(man1_MANS:.1=.txt) \
	$(man5_MANS:.5=.txt) \
	$(man7_MANS:.7=.txt) \
	$(MANS_ALL)

if ENABLE_DOCS_BUILD

DISTCLEANFILES += $(MANS_ALL)

%.1 %.5 %.7: %.txt docs/shared-options.txt docs/remote-paths.txt docs/footer.txt docs/asciidoc.conf
	$(AM_V_GEN)a2x --asciidoc-opts="-f docs/asciidoc.conf" -f manpage $<
	@sed -i 's/\[FIXME: manual]/Megatools Manual/' $@
	@sed -i 's/\[FIXME: source]/megatools $(VERSION)/' $@

else

%.1 %.5 %.7:
	@echo You need to use --enable-docs-build if building from git tree
	@false

endif

# }}}
# {{{ tests

if ENABLE_TESTS
noinst_PROGRAMS = tests/test-aes tests/test-rsa
endif

tests_test_aes_SOURCES = tests/test-aes.c
tests_test_rsa_SOURCES = tests/test-rsa.c

EXTRA_DIST += \
	tests/config.js \
	tests/test-bindings.js \
	tests/test-bindings.py \
	tests/test-bindings.lua \
	tests/test-bindings.rb \
	tests/test-bindings.sh \
	tests/test-server.sh \
	tests/test-server.js

EXTRA_DIST += LICENSE Makefile.glib HACKING

doc_DATA = LICENSE NEWS TODO README INSTALL HACKING

# }}}

pkgconfigdir = $(libdir)/pkgconfig
pkgconfig_DATA = libmega.pc

export MAKEFLAGS += --no-print-directory -s
ACLOCAL_AMFLAGS = -I m4
