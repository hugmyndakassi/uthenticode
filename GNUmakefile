.DEFAULT: build

ifeq ($(CFG),)
CFG:=DEBUG
endif

COMPILE.cpp = $(CXX) $(CXXFLAGS) $(CXXFLAGS.$(CFG)) $(CPPFLAGS) $(CPPFLAGS.$(CFG)) $(TARGET_ARCH) -c
LINK.cpp = $(CXX) $(LDFLAGS) $(TARGET_ARCH)

SOURCES.PEPARSE:=$(filter-out %/unicode_winapi.cpp,$(wildcard pe-parse/pe-parser-library/src/*.cpp))
SOURCES.UTHENTICODE:=$(wildcard src/*.cpp)
SOURCES.SVCLI:=$(wildcard src/svcli/*.cpp)
VERSION.PEPARSE:=$(shell cat pe-parse/VERSION)
VERSION.UTHENTICODE:=$(shell cat VERSION)
OBJECTS.PEPARSE:=$(addprefix obj/,$(SOURCES.PEPARSE:.cpp=.o))
OBJECTS.UTHENTICODE:=$(addprefix obj/,$(SOURCES.UTHENTICODE:.cpp=.o))
OBJECTS.SVCLI:=$(addprefix obj/,$(SOURCES.SVCLI:.cpp=.o))
OBJECTS:=$(OBJECTS.PEPARSE) $(OBJECTS.UTHENTICODE) $(OBJECTS.UTHENTICODE)

CPPFLAGS.RELEASE:=-DNDEBUG
CXXFLAGS.RELEASE:=-O3
CPPFLAGS.DEBUG:=-DDEBUG -D_DEBUG
CXXFLAGS.DEBUG:=-O0

LDLIBS:= \
			-l:libcrypto.a
CPPFLAGS:= \
			-MMD
CPPFLAGS.UTHENTICODE:= \
			-DUTHENTICODE_VERSION=\"$(VERSION.UTHENTICODE)\" \
			-Isrc/include
CPPFLAGS.PEPARSE:= \
			-DPEPARSE_VERSION=\"$(VERSION.PEPARSE)\" \
			-Ipe-parse/pe-parser-library/include
CXXFLAGS:= \
			-g3 \
			-ggdb \
			-fPIC \
			-std=c++17 \
			-Wall \
			-Wextra \
			-pedantic \
			-Werror
CXXFLAGS.PEPARSE:= \
			-Wcast-align \
			-Wcast-qual \
			-Wcomment \
			-Wctor-dtor-privacy \
			-Wdisabled-optimization \
			-Wformat=2 \
			-Winit-self \
			-Wlong-long \
			-Wmissing-declarations \
			-Wmissing-include-dirs \
			-Wno-missing-declarations \
			-Wno-strict-overflow \
			-Wold-style-cast \
			-Woverloaded-virtual \
			-Wredundant-decls \
			-Wshadow \
			-Wsign-conversion \
			-Wsign-promo \
			-Wstrict-overflow=5 \
			-Wswitch-default \
			-Wundef \
			-Wuninitialized \
			-Wunused
ifneq ($(VERBOSE),)
LDFLAGS:= $(LDFLAGS) \
			-Wl,--verbose
CXXFLAGS:= $(CXXFLAGS) \
			--verbose
endif

$(OBJECTS.PEPARSE): CXXFLAGS+=$(CXXFLAGS.PEPARSE)
$(OBJECTS.PEPARSE): CPPFLAGS+=$(CPPFLAGS.PEPARSE)
$(OBJECTS.UTHENTICODE) $(OBJECTS.SVCLI): CPPFLAGS+=$(CPPFLAGS.UTHENTICODE) $(filter -I%,$(CPPFLAGS.PEPARSE))
$(OBJECTS.UTHENTICODE) $(SOURCES.PEPARSE): pe-parse

obj/%.o: %.cpp
	@test -d $(@D) || mkdir -p $(@D)
	$(strip $(COMPILE.cpp) $(OUTPUT_OPTION) $<)

svcli: $(OBJECTS.PEPARSE) $(OBJECTS.UTHENTICODE) $(OBJECTS.SVCLI)
	$(strip $(LINK.cpp) $^ $(LOADLIBES) $(LDLIBS) -o $@)

libpe-parse.so: $(OBJECTS.PEPARSE) $(OBJECTS.UTHENTICODE)
	$(strip $(LINK.cpp) $^ $(LOADLIBES) $(LDLIBS) -o $@)

libpe-parse.so: LDFLAGS+=-shared -Wl,-soname,libpe-parse.so

build: svcli libpe-parse.so

include ./Makefile

pe-parse::
	test -d $@ || git clone https://github.com/trailofbits/pe-parse.git $@
	git -C $@ pull --all

clean:
	rm -f -- svcli libpe-parse.so
	rm -rf -- build obj svcli

mrproper: clean
	rm -rf -- pe-parse

rebuild: clean build

.PHONY: build clean mrproper pe-parse rebuild
.NOTPARALLEL: pe-parse

-include $(OBJECTS:.o=.d)
