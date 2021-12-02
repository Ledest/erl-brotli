# Based on c_src.mk from erlang.mk by Loic Hoguin <essen@ninenines.eu>

CURDIR := $(shell pwd)
BASEDIR := $(abspath $(CURDIR)/..)

PROJECT = brotli

ERTS_INCLUDE_DIR ?= $(shell erl -noshell -s init stop -eval "io:format(\"~s/erts-~s/include/\", [code:root_dir(), erlang:system_info(version)]).")

OUT ?= $(BASEDIR)/priv

# System type and C compiler/flags.
#

UNAME_SYS := $(shell uname -s)
ifeq ($(UNAME_SYS), Darwin)
	CC ?= cc
	CFLAGS += -O2 -std=c99 -arch x86_64 -finline-functions -Wall
	LDFLAGS ?= -arch x86_64 -flat_namespace -undefined suppress
else ifeq ($(UNAME_SYS), FreeBSD)
	CC = /usr/bin/clang
	CFLAGS += -O2 -std=c99 -finline-functions -Wall
else ifeq ($(UNAME_SYS), Linux)
	CC ?= gcc
	CFLAGS += -O2 -std=c99 -finline-functions -Wall
endif

ifneq ($(LIBBROTLI_DIR),)
BROTLI_CFLAGS = -I $(LIBBROTLI_DIR)/c/include
BROTLI_STATIC_LIBS = $(LIBBROTLI_DIR)/libbrotlidec-static.a $(LIBBROTLI_DIR)/libbrotlienc-static.a $(LIBBROTLI_DIR)/libbrotlicommon-static.a
BROTLI_LIBS = $(BROTLI_STATIC_LIBS)
else
BROTLI_CFLAGS = $(shell pkg-config --cflags libbrotlidec) \
       $(shell pkg-config --cflags libbrotlienc) \
       $(shell pkg-config --cflags libbrotlicommon)
BROTLI_STATIC_LIBS =
BROTLI_LIBS = $(shell pkg-config --libs libbrotlidec) \
       $(shell pkg-config --libs libbrotlienc) \
       $(shell pkg-config --libs libbrotlicommon)
endif

CFLAGS += -fPIC -I $(ERTS_INCLUDE_DIR) $(BROTLI_CFLAGS)

LDFLAGS += -shared -rdynamic

LDLIBS += $(BROTLI_LIBS)

# Verbosity.

c_verbose_0 = @echo " C     " $(?F);
c_verbose = $(c_verbose_$(V))

cpp_verbose_0 = @echo " CPP   " $(?F);
cpp_verbose = $(cpp_verbose_$(V))

link_verbose_0 = @echo " LD    " $(@F);
link_verbose = $(link_verbose_$(V))

lib: $(OUT)/$(PROJECT).so

$(OUT)/$(PROJECT).so: brotli_nif.o $(BROTLI_STATIC_LIBS)
	@mkdir -p $(@D)
	$(link_verbose) $(CC) $< $(LDFLAGS) -o $@ $(LDLIBS)

%.o: %.c
	$(c_verbose) $(CC) $(CFLAGS) $(CPPFLAGS) -c $< -o $@

$(BROTLI_STATIC_LIBS):
	cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_FLAGS_RELEASE="-O2 -DNDEBUG" -S $(LIBBROTLI_DIR) -B $(LIBBROTLI_DIR)
	cmake --build $(LIBBROTLI_DIR) -j --config Release --verbose --target brotlicommon-static/fast brotlienc-static/fast brotlidec-static/fast

clean:
	@rm -f $(OUT)/$(PROJECT).so brotli_nif.o
	@$(MAKE) -C $(LIBBROTLI_DIR) clean

.PHONY: lib
