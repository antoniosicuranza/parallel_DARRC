CC=gcc
LD=gcc
LDFLAGS=
LIBS=-lbft -ljemalloc -DJUDYERROR_NOTEST -lJudy  -lm
CFLAGS=-O3 -Wall -std=c99 -Wno-unknown-pragmas -D_7ZIP_ST

DESTDIR		?=
PREFIX 		?= /usr/local
BINDIR 		= $(PREFIX)/bin

DARRC_SRC_DIR	= src
DARRC_INC_DIR	= include
DARRC_CPPFLAGS	= -I$(DARRC_INC_DIR) -I$(LZMA_DIR) -I$(ZSTDPROGDIR)
DARRC_FLAGS	= $(DARRC_CPPFLAGS) $(CFLAGS)

LZMA_DIR = external/lzma

ZSTDLIBDIR = external/zstd/lib
ZSTDPROGDIR = external/zstd/programs

ifeq ($(shell $(CC) -v 2>&1 | grep -c "gcc version "), 1)
ALIGN_LOOP = -falign-loops=32
else
ALIGN_LOOP =
endif

ZSTD_CPPFLAGS= -I$(ZSTDLIBDIR) -I$(ZSTDLIBDIR)/common -I$(ZSTDLIBDIR)/dictBuilder
ZSTD_CFLAGS ?= -O3
ZSTD_CFLAGS += -Wall -Wextra -Wcast-qual -Wcast-align -Wshadow -Wstrict-aliasing=1 \
          -Wswitch-enum -Wdeclaration-after-statement -Wstrict-prototypes -Wundef
ZSTD_FLAGS   = $(ZSTD_CPPFLAGS) $(ZSTD_CFLAGS)

ZSTD_COMMON_O := $(ZSTDLIBDIR)/common/fse_decompress.o $(ZSTDLIBDIR)/common/entropy_common.o \
		$(ZSTDLIBDIR)/common/xxhash.o $(ZSTDLIBDIR)/common/zstd_common.o
ZSTD_COMP_O := $(ZSTDLIBDIR)/compress/zstd_compress.o $(ZSTDLIBDIR)/compress/fse_compress.o \
		$(ZSTDLIBDIR)/compress/huf_compress.o
ZSTD_HUFDECOMP_O := $(ZSTDLIBDIR)/decompress/huf_decompress.o
ZSTD_DICT_O := $(ZSTDLIBDIR)/dictBuilder/divsufsort.o $(ZSTDLIBDIR)/dictBuilder/zdict.o
ZSTD_PROG_O := $(ZSTDPROGDIR)/fileio.o

ZSTD_DECOMP_O = $(ZSTDLIBDIR)/decompress/zstd_decompress.o

ifeq ($(ZSTD_LEGACY_SUPPORT), 0)
ZSTD_CPPFLAGS += -DZSTD_LEGACY_SUPPORT=0
ZSTD_LEGACY_O :=
else
ZSTD_LEGACY_SUPPORT:=1
ZSTD_CPPFLAGS += -I$(ZSTDLIBDIR)/legacy
ZSTD_LEGACY_O := $(ZSTDLIBDIR)/legacy/zstd_v01.o $(ZSTDLIBDIR)/legacy/zstd_v02.o $(ZSTDLIBDIR)/legacy/zstd_v03.o \
		$(ZSTDLIBDIR)/legacy/zstd_v04.o $(ZSTDLIBDIR)/legacy/zstd_v05.o $(ZSTDLIBDIR)/legacy/zstd_v06.o \
		$(ZSTDLIBDIR)/legacy/zstd_v07.o
endif

ZSTD_FILES_O := $(ZSTD_HUFDECOMP_O) $(ZSTD_COMMON_O) $(ZSTD_COMP_O) $(ZSTD_LEGACY_O) $(ZSTD_DICT_O) $(ZSTD_PROG_O)

LZMA_FILES_O := $(LZMA_DIR)/7zFile.o $(LZMA_DIR)/7zStream.o $(LZMA_DIR)/Alloc.o $(LZMA_DIR)/LzFind.o $(LZMA_DIR)/LzmaDec.o $(LZMA_DIR)/LzmaEnc.o

DARRC_FILES_O := $(DARRC_SRC_DIR)/compression.o $(DARRC_SRC_DIR)/lz_utils.o $(DARRC_SRC_DIR)/main.o

all: darrc clean_files

darrc: $(ZSTD_DECOMP_O) $(ZSTD_FILES_O) $(LZMA_FILES_O) $(DARRC_FILES_O) -lpthread
	$(LD) -o $@ $^ $(LDFLAGS) $(LIBS)

$(ZSTD_DECOMP_O): %.o: %.c
	$(CC) $(ALIGN_LOOP) $(ZSTD_FLAGS) -DZSTD_LEGACY_SUPPORT=$(ZSTD_LEGACY_SUPPORT) -c $< -o $@

$(ZSTD_FILES_O): %.o: %.c
	$(CC) $(ZSTD_FLAGS) -DZSTD_LEGACY_SUPPORT=$(ZSTD_LEGACY_SUPPORT) -c $< -o $@

$(LZMA_FILES_O): %.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

$(DARRC_FILES_O): %.o: %.c
	$(CC) $(DARRC_FLAGS) -c $< -o $@

install:
	@install -m 755 darrc $(DESTDIR)$(BINDIR)/darrc
	@echo DARRC installation finished

clean_files: 
	rm -f $(ZSTD_DECOMP_O) $(ZSTD_FILES_O) $(LZMA_FILES_O) $(DARRC_FILES_O)

clean:	clean_files
	rm -f darrc

.PHONY: clean

