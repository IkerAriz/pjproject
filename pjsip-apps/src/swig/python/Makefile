PYTHON_SO=_pjsua2.so

USE_PYTHON3?=1
USE_SWIG_PREGEN?=0

ifeq ($(USE_PYTHON3),1)
  PYTHON_EXE=python3
else
  PYTHON_EXE=python2
endif

#PYTHON_SETUP_FLAGS = --inplace 
ifeq ($(OS),Windows_NT)
  PYTHON_PKG_DIR := $(shell $(PYTHON_EXE) -m site --user-site | sed 's/\\/\//g')
  PYTHON_SETUP_FLAGS += --compiler=mingw32
  GCC_EXE=gcc.exe
else
  PYTHON_PKG_DIR := $(shell $(PYTHON_EXE) -m site --user-site)
  GCC_EXE=
endif

SWIG_FLAGS=-I../../../../pjlib/include \
		   -I../../../../pjlib-util/include \
		   -I../../../../pjmedia/include \
		   -I../../../../pjsip/include \
		   -I../../../../pjnath/include -c++ 
SRC_DIR=../../../../pjsip/include
SRCS=$(SRC_DIR)/pjsua2/endpoint.hpp $(SRC_DIR)/pjsua2/types.hpp

GEN_DOC = -doxygen
USE_THREADS = -threads
# In SWIG 4.2 the build will fail if we use this flag
#-DSWIG_NO_EXPORT_ITERATOR_METHODS
SWIG_FLAGS += -w312 $(USE_THREADS) $(GEN_DOC)

.PHONY: all install uninstall

all: $(PYTHON_SO)

wheel: pjsua2_wrap.cpp setup.py $(GCC_EXE)
	$(PYTHON_EXE) setup.py bdist_wheel $(PYTHON_SETUP_FLAGS)

$(PYTHON_SO): pjsua2_wrap.cpp setup.py $(GCC_EXE)
	$(PYTHON_EXE) setup.py build $(PYTHON_SETUP_FLAGS)

gcc.exe: cc_mingw.c
	gcc -o gcc.exe cc_mingw.c
	cp gcc.exe g++.exe


ifeq ($(USE_SWIG_PREGEN),1)

# useful for projects that vendor pjsip and want to allow
# building the whole project without swig; these projects
# can pregenerate the swig output and include it along
# with their vendored copy of pjsip

pjsua2_wrap.cpp: use_pregen
pjsua2_wrap.h: use_pregen
pjsua2.py: use_pregen

use_pregen:
	@test -n "${SWIG_PREGEN_DIR}" || (echo "ERROR: SWIG_PREGEN_DIR is not defined" && exit 1)
	@test -d "${SWIG_PREGEN_DIR}" || (echo "ERROR: SWIG_PREGEN_DIR does not point to a directory (${SWIG_PREGEN_DIR})" && exit 1)
	cp ${SWIG_PREGEN_DIR}/pjsua2_wrap.cpp pjsua2_wrap.cpp
	cp ${SWIG_PREGEN_DIR}/pjsua2_wrap.h pjsua2_wrap.h
	cp ${SWIG_PREGEN_DIR}/pjsua2.py pjsua2.py

else

pjsua2_wrap.cpp: ../pjsua2.i ../symbols.i Makefile $(SRCS)
	swig $(SWIG_FLAGS) -python -o pjsua2_wrap.cpp ../pjsua2.i

endif

clean distclean realclean:
	rm -rf $(PYTHON_SO) pjsua2_wrap.cpp pjsua2_wrap.h pjsua2.py build *.pyc
	rm -f gcc.exe g++.exe

install:
	$(PYTHON_EXE) setup.py install --user

uninstall:
	rm -f $(PYTHON_PKG_DIR)/pjsua2*
	rm -f $(PYTHON_PKG_DIR)/_pjsua2*
