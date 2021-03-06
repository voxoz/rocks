# Barrel dependecy Makefile

UNAME_SYS := $(shell sh -c 'uname -s 2>/dev/null || echo not')
ifeq ($(UNAME_SYS), Darwin)
	CC ?= cc
	CFLAGS ?= -O2 -std=c99 -Wall -Wmissing-prototypes
	CXXFLAGS ?= -O2
	#LDFLAGS ?= -arch x86_64 -flat_namespace -undefined suppress
else ifeq ($(UNAME_SYS), FreeBSD)
	CC = clang
	CXX = clang++
	CFLAGS ?= -O2 -D_GCLIBCXX_USE_C99
	CXXFLAGS ?= -O2 -std=c++11 -stdlib=libc++ -D_GCLIBCXX_USE_C99
else ifeq ($(UNAME_SYS), OpenBSD)
	CC = egcc
	CXX = eg++
	CFLAGS ?= -O2 -std=c99 -finline-functions -Wall -Wmissing-prototypes
	CFLAGS += -D_GCLIBCXX_USE_C99
	CXXFLAGS ?= -O2
	CXXFLAGS += -std=c++11 -pthread -finline-functions -D_GCLIBCXX_USE_C99
else ifeq ($(UNAME_SYS), DragonFly)
	CC = c++48
	CXX = c++48
	CFLAGS ?= -O2 -std=c99 -finline-functions -Wall -Wmissing-prototypes
	CFLAGS += -D_GCLIBCXX_USE_C99
	CXXFLAGS ?= -O2 -finline-functions
	CXXFLAGS += -D_GCLIBCXX_USE_C99
else ifeq ($(UNAME_SYS), NetBSD)
	CC = c++48
	CXX = c++48
	CFLAGS ?= -O2 -std=c99 -finline-functions -Wall -Wmissing-prototypes
	CFLAGS += -D_GCLIBCXX_USE_C99
	CXXFLAGS ?= -O2 -finline-functions
	CXXFLAGS += -D_GCLIBCXX_USE_C99
else ifeq ($(UNAME_SYS), SunOS)
	ifeq ($(shell uname -v | sed 's/_.*$$//'), joyent)
		CC = gcc
		CXX = g++
	else
		CC = c++48
		CXX = c++48
	endif
	CFLAGS ?= -O2 -std=c99 -finline-functions -Wall -Wmissing-prototypes
	CFLAGS += -D_GCLIBCXX_USE_C99
	CXXFLAGS ?= -O2 -finline-functions
	CXXFLAGS += -D_GCLIBCXX_USE_C99
else ifeq ($(UNAME_SYS), Linux)
	CC ?= gcc
	CFLAGS ?= -O2 -std=c99 -finline-functions -Wall -Wmissing-prototypes
	CXXFLAGS ?= -O2 -finline-functions
endif

SNAPPYLIB=$(CURDIR)/snappy/build/libsnappy.a
LZ4LIB=$(CURDIR)/lz4/lib/liblz4.a
RDBLIB=$(CURDIR)/rocksdb/librocksdb.a

RDB_CXXFLAGS="$(CXXFLAGS) -DNDEBUG -DSNAPPY -DLZ4 -I$(CURDIR)/snappy/build -I$(CURDIR)/snappy -I$(CURDIR)/lz4/lib -fPIC"
RDB_CFLAGS="$(CFLAGS) -DNDEBUG -I$(CURDIR)/snappy/build -I$(CURDIR)/snappy -I$(CURDIR)/lz4/lib"
RDB_LDFLAGS="$(SNAPPYLIB) $(LZ4LIB) $(LDFLAGS) -L$(CURDIR)/snappy/build -L$(CURDIR)/lz4/lib"

ifeq ($(UNAME_SYS),OpenBSD)
	RDB_CXXFLAGS="$(RDB_CXXFLAGS) -DOS_OPENBSD"
	RDB_CFLAGS="$(RDB_CFLAGS) -DOS_OPENBSD"
	RDB_LDFLAGS="$(LDFLAGS) -lstdc++"
	RDB_CPPFLAGS="-DOS_OPENBSD"
endif



default:
	@echo "Explicit target required"

.PHONY: default

distclean:
	@-(cd rocksdb && $(MAKE) clean) > /dev/null || true
	@-(cd snappy && rm -rf build) > /dev/null || true
	@-(cd lz4 && $(MAKE) clean) > /dev/null || true

.PHONT: distclean

$(SNAPPYLIB):
	@mkdir -p snappy/build
	@cd snappy/build && \
		cmake ../ -DCMAKE_POSITION_INDEPENDENT_CODE=ON && $(MAKE)

.PHONY: snappy

$(LZ4LIB):
	@cd lz4 && $(MAKE) CFLAGS="-O2 -fPIC" CXX="$(CXX)" 

.PHONY: lz4

rocksdb: $(RDBLIB)

$(RDBLIB): $(SNAPPYLIB) $(LZ4LIB)
	@cd rocksdb && \
		CXX=$(CXX) CC=$(CC) $(MAKE) static_lib EXTRA_CXXFLAGS=$(RDB_CXXFLAGS) EXTRA_LDFLAGS=$(RDB_LDFLAGS) USE_RTTI=1 DISABLE_JEMALLOC=1 ROCKSDB_DISABLE_ZSTD=1 DEBUG_LEVEL=0 PORTABLE=1

.PHONY: rocksdb

