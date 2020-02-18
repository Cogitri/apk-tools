PKG_CONFIG ?= pkg-config
LUAAPK ?= yes

OPENSSL_CFLAGS		:= $(shell $(PKG_CONFIG) --cflags openssl)
OPENSSL_LIBS		:= $(shell $(PKG_CONFIG) --libs openssl)

ZLIB_CFLAGS		:= $(shell $(PKG_CONFIG) --cflags zlib)
ZLIB_LIBS		:= $(shell $(PKG_CONFIG) --libs zlib)

# lua module
ifneq ($(LUAAPK),)
LUA_VERSION		?= 5.3
LUA_PC			?= lua$(LUA_VERSION)
LUA_LIBDIR		?= /usr/lib/lua/$(LUA_VERSION)
shlibs-y		+= apk.so
apk.so-objs		:= lua-apk.o
CFLAGS_lua-apk.o	:= -DAPK_VERSION=\"$(FULL_VERSION)\" \
			   $(shell $(PKG_CONFIG) $(LUA_PC) --cflags)
LUA_LIB-y		:= $(obj)/apk.so
install-LUA_LIB-y	:= $(INSTALLDIR) $(DESTDIR)$(LUA_LIBDIR) && \
			   $(INSTALL) $(LUA_LIB-y) $(DESTDIR)$(LUA_LIBDIR)
endif

progs-y			+= apk
apk-objs		:= apk.o add.o del.o fix.o update.o info.o list.o \
			   search.o upgrade.o cache.o ver.o index.o fetch.o \
			   audit.o verify.o dot.o policy.o stats.o manifest.o

libapk.so.$(VERSION)-objs		:= common.o database.o package.o archive.o \
			   version.o io.o url.o gunzip.o blob.o hash.o print.o \
			   commit.o solver.o
libapk.so.$(VERSION)-libs		:= libfetch/libfetch.a

libapk.a-objs		:= $(libapk.so.$(VERSION)-objs)
libapk.a-libs		:= $(libapk.so.$(VERSION)-libs)

libs-y += libapk.a
LIBAPK_STATIC-y	 = $(obj)/libapk.a
install-LIBAPK_STATIC-y := $(INSTALLDIR) $(DESTDIR)$(LIBDIR) && \
			   $(INSTALL) $(LIBAPK_STATIC-y) $(DESTDIR)$(LIBDIR)
apk-static-libs := $(libapk.a-libs) $(obj)/libapk.a

libapk_soname := $(basename $(basename $(VERSION)))
shlibs-y += libapk.so.$(VERSION)
LDFLAGS_libapk.so.$(VERSION) += -Wl,-soname,libapk.so.$(libapk_soname)
LIBAPK_SHARED-y = $(obj)/libapk.so.$(VERSION)
install-LIBAPK_SHARED-y := $(INSTALLDIR) $(DESTDIR)$(LIBDIR) && \
			   $(INSTALL) $(LIBAPK_SHARED-y) $(DESTDIR)$(LIBDIR)/libapk.so.$(VERSION) && \
			   $(INSTALL) libapk.so.$(libapk_soname) $(DESTDIR)$(LIBDIR) && \
			   $(INSTALL) libapk.so $(DESTDIR)$(LIBDIR)

ifeq ($(TEST),y)
progs-y += apk-test
apk-test-objs := apk-test.o $(filter-out apk.o, $(apk-objs))
$(obj)/apk-test: $(LIBAPK_SHARED-y)
endif

LIBS_apk := -lapk
LIBS_apk-test := -lapk
LIBS_apk.so := -L$(obj) -lapk

CFLAGS_ALL		+= -D_ATFILE_SOURCE -Ilibfetch
CFLAGS_apk.o		:= -DAPK_VERSION=\"$(FULL_VERSION)\"
CFLAGS_apk-static.o	:= -DAPK_VERSION=\"$(FULL_VERSION)\" -DOPENSSL_NO_ENGINE
CFLAGS_apk-test.o	:= -DAPK_VERSION=\"$(FULL_VERSION)\" -DOPENSSL_NO_ENGINE -DTEST_MODE

progs-$(STATIC)		+= apk.static
apk.static-objs		:= $(filter-out apk.o,$(apk-objs)) apk-static.o
apk.static-libs		:= $(apk-static-libs)
LDFLAGS_apk.static	:= -static
LIBS_apk.static		:= -Wl,--as-needed -ldl -Wl,--no-as-needed
LDFLAGS_apk		+= -L$(obj)
LDFLAGS_apk-test	+= -L$(obj)

CFLAGS_ALL		+= $(OPENSSL_CFLAGS) $(ZLIB_CFLAGS)
LIBS			:= -Wl,--as-needed \
				$(OPENSSL_LIBS) $(ZLIB_LIBS) \
			   -Wl,--no-as-needed

$(obj)/apk: $(LIBAPK_SHARED-y)

$(obj)/apk.so: $(LIBAPK_SHARED-y)

$(obj)/apk.a: $(LIBAPK_STATIC-y)

$(obj)/libapk.so.$(libapk_soname):
	$(LN) libapk.so.$(VERSION) $(obj)/libapk.so.$(libapk_soname)

$(obj)/libapk.so:
	$(LN) libapk.so.$(libapk_soname) $(obj)/libapk.so

generate-y	+= libapk.so.$(libapk_soname) libapk.so

generate-y	+= apk.pc

$(obj)/apk.pc:
	cp src/apk.pc.in $(obj)/apk.pc
	sed -e "s|@EXEC_DIR@|$(SBINDIR)|" \
	-e "s|@LIB_DIR@|$(LIBDIR)|" \
	-e "s|@INCLUDE_DIR@|$(INCLUDEDIR)|" \
	-e "s|@VERSION@|$(VERSION)|" -i $(obj)/apk.pc

install: $(obj)/apk $(LIBAPK-y) $(LUA_LIB-y)
	$(INSTALLDIR) $(DESTDIR)$(SBINDIR)
	$(INSTALL) $(obj)/apk $(DESTDIR)$(SBINDIR)
	$(install-LIBAPK_SHARED-y)
	$(install-LIBAPK_STATIC-y)
	$(install-LUA_LIB-y)
	$(INSTALLDIR) $(DESTDIR)$(INCLUDEDIR)
	$(INSTALL) -m644 src/*.h $(DESTDIR)$(INCLUDEDIR)
	$(INSTALLDIR) $(DESTDIR)$(PKGCONFIGDIR)
	$(INSTALL) -m644 $(obj)/apk.pc $(DESTDIR)$(PKGCONFIGDIR)

