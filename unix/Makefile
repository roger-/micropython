include ../py/mkenv.mk
-include mpconfigport.mk

# define main target
PROG = micropython

# qstr definitions (must come before including py.mk)
QSTR_DEFS = qstrdefsport.h

# OS name, for simple autoconfig
UNAME_S := $(shell uname -s)

# include py core make definitions
include ../py/py.mk

INC =  -I.
INC += -I$(PY_SRC)
INC += -I$(BUILD)

# compiler settings
CWARN = -Wall -Werror -Wno-error=cpp
CFLAGS = $(INC) $(CWARN) -ansi -std=gnu99 -DUNIX $(CFLAGS_MOD) $(COPT) $(CFLAGS_EXTRA)

# Debugging/Optimization
ifdef DEBUG
CFLAGS += -g
COPT = -O0
else
COPT = -Os #-DNDEBUG
endif

LDFLAGS = $(LDFLAGS_MOD) -lm -Wl,-Map=$@.map,--cref $(LDFLAGS_EXTRA)

ifeq ($(MICROPY_FORCE_32BIT),1)
CFLAGS += -m32
LDFLAGS += -m32
ifeq ($(MICROPY_PY_FFI),1)
ifeq ($(UNAME_S),Linux)
CFLAGS_MOD += -I/usr/include/i686-linux-gnu
endif
endif
endif

ifeq ($(MICROPY_USE_READLINE),1)
CFLAGS_MOD += -DMICROPY_USE_READLINE=1
LDFLAGS_MOD += -lreadline
# the following is needed for BSD
#LDFLAGS_MOD += -ltermcap
endif
ifeq ($(MICROPY_PY_TIME),1)
CFLAGS_MOD += -DMICROPY_PY_TIME=1
SRC_MOD += modtime.c
endif
ifeq ($(MICROPY_PY_TERMIOS),1)
CFLAGS_MOD += -DMICROPY_PY_TERMIOS=1
SRC_MOD += modtermios.c
endif
ifeq ($(MICROPY_PY_FFI),1)
LIBFFI_LDFLAGS_MOD := $(shell pkg-config --libs libffi)
LIBFFI_CFLAGS_MOD := $(shell pkg-config --cflags libffi)
CFLAGS_MOD += $(LIBFFI_CFLAGS_MOD) -DMICROPY_PY_FFI=1
ifeq ($(UNAME_S),Linux)
LDFLAGS_MOD += -ldl
endif
LDFLAGS_MOD += $(LIBFFI_LDFLAGS_MOD)
SRC_MOD += modffi.c
endif


# source files
SRC_C = \
	main.c \
	gccollect.c \
	input.c \
	file.c \
	modsocket.c \
	modos.c \
	$(SRC_MOD)

ifeq ($(UNAME_S),Darwin)

LDFLAGS+ = -Wl,-order_file,$(BUILD)/order.def

# Must be the last file in list of sources
SRC_C += seg_helpers.c

# making seg_helpers.c rely on order.def will force order.def to be created
seg_helpers.c: $(BUILD)/order.def

# create order.def in build directory
$(BUILD)/order.def:
	$(Q)echo "seg_helpers.o: ___bss_start" > $@
endif
	
OBJ = $(PY_O) $(addprefix $(BUILD)/, $(SRC_C:.c=.o))

include ../py/mkrules.mk

.PHONY: test

test: $(PROG) ../tests/run-tests
	$(eval DIRNAME=$(notdir $(CURDIR)))
	cd ../tests && MICROPY_MICROPYTHON=../$(DIRNAME)/$(PROG) ./run-tests

# install micropython in /usr/local/bin
TARGET = micropython
PREFIX = $(DESTDIR)/usr/local
BINDIR = $(PREFIX)/bin
PIPSRC = ../tools/pip-micropython
PIPTARGET = pip-micropython

install: micropython
	install -D $(TARGET) $(BINDIR)/$(TARGET)
	install -D $(PIPSRC) $(BINDIR)/$(PIPTARGET)

# uninstall micropython
uninstall:
	-rm $(BINDIR)/$(TARGET)
	-rm $(BINDIR)/$(PIPTARGET)
