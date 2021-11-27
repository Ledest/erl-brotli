# Based on c_src.mk from erlang.mk by Loic Hoguin <essen@ninenines.eu>

CURDIR := $(shell pwd)
BASEDIR := $(abspath $(CURDIR)/..)

PROJECT = brotli

ERTS_INCLUDE_DIR ?= $(shell erl -noshell -s init stop -eval "io:format(\"~s/erts-~s/include/\", [code:root_dir(), erlang:system_info(version)]).")

ifndef REBAR_BARE_COMPILER_OUTPUT_DIR
	OUT ?= $(BASEDIR)/priv
else
	OUT ?= $(REBAR_BARE_COMPILER_OUTPUT_DIR)/priv
endif

# System type and C compiler/flags.
#

UNAME_SYS := $(shell uname -s)
ifeq ($(UNAME_SYS), Darwin)
	CC ?= cc
	CFLAGS += -O2 -std=c99 -finline-functions -Wall
	LDFLAGS ?= -flat_namespace -undefined suppress
else ifeq ($(UNAME_SYS), FreeBSD)
	CC = /usr/bin/clang
	CFLAGS += -O2 -std=c99 -finline-functions -Wall
else ifeq ($(UNAME_SYS), Linux)
	CC ?= gcc
	CFLAGS += -O2 -std=c99 -finline-functions -Wall
endif

CFLAGS += -fPIC -I $(ERTS_INCLUDE_DIR) \
	$(shell pkg-config --cflags libbrotlidec) \
	$(shell pkg-config --cflags libbrotlienc) \
	$(shell pkg-config --cflags libbrotlicommon)

LDFLAGS += -shared -rdynamic

LDLIBS += $(shell pkg-config --libs libbrotlidec) \
	$(shell pkg-config --libs libbrotlienc) \
	$(shell pkg-config --libs libbrotlicommon)

# Verbosity.

c_verbose_0 = @echo " C     " $(?F);
c_verbose = $(c_verbose_$(V))

cpp_verbose_0 = @echo " CPP   " $(?F);
cpp_verbose = $(cpp_verbose_$(V))

link_verbose_0 = @echo " LD    " $(@F);
link_verbose = $(link_verbose_$(V))

lib: $(OUT)/$(PROJECT).so

$(OUT)/$(PROJECT).so: brotli_nif.o
	@mkdir -p $(@D)
	$(link_verbose) $(CC) $^ $(LDFLAGS) -o $@ $(LDLIBS)

%.o: %.c
	$(c_verbose) $(CC) $(CFLAGS) $(CPPFLAGS) -c $< -o $@

clean:
	@rm -f $(OUT)/$(PROJECT).so brotli_nif.o

.PHONY: lib
