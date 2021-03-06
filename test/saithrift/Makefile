CXX=g++
SAI_PREFIX = /usr
SAI_HEADER_DIR ?= $(SAI_PREFIX)/include/sai
SAI_HEADERS = $(SAI_HEADER_DIR)/sai*.h
CFLAGS = -I$(SAI_HEADER_DIR) -I. -std=c++11
ifeq ($(DEBUG),1)
CFLAGS += -O0 -ggdb
endif

ifeq ($(platform),MLNX)
CDEFS = -DMLNXSAI
else
ifeq ($(platform),BFT)
CDEFS = -DBFTSAI
else
ifeq ($(platform),CAVIUM)
CDEFS = -DCAVIUMSAI
else
CDEFS = -DBRCMSAI
endif
endif
endif
DEPS = switch_sai_constants.h  switch_sai_rpc.h  switch_sai_types.h
OBJS = switch_sai_constants.o  switch_sai_rpc.o  switch_sai_types.o

ODIR = ./src/obj
SAIDIR = ./include
SRC = ./src
THRIFT = /usr/bin/thrift
CTYPESGEN = /usr/local/bin/ctypesgen.py
LIBS = -lthrift -lpthread  -lsai
SAI_LIBRARY_DIR ?= $(SAI_PREFIX)/lib
LDFLAGS = -L$(SAI_LIBRARY_DIR) -Wl,-rpath=$(SAI_LIBRARY_DIR)
CPP_SOURCES = \
				src/gen-cpp/switch_sai_constants.cpp \
				src/gen-cpp/switch_sai_constants.h \
				src/gen-cpp/switch_sai_rpc.cpp \
				src/gen-cpp/switch_sai_rpc.h \
				src/gen-cpp/switch_sai_types.cpp \
				src/gen-cpp/switch_sai_types.h

PY_SOURCES = \
				src/gen-py/__init__.py \
				src/gen-py/switch_sai/constants.py \
				src/gen-py/switch_sai/__init__.py \
				src/gen-py/switch_sai/switch_sai_rpc.py \
				src/gen-py/switch_sai/switch_sai_rpc-remote \
				src/gen-py/switch_sai/ttypes.py

SAI_PY_HEADERS = \
                src/gen-py/switch_sai/sai_headers.py

MKDIR_P = mkdir -p
INSTALL := /usr/bin/install

all: directories $(ODIR)/librpcserver.a saiserver clientlib

directories:
	$(MKDIR_P) $(ODIR)

$(CPP_SOURCES): src/switch_sai.thrift
	$(THRIFT) -o $(SRC) --gen cpp -r $(SRC)/switch_sai.thrift

$(PY_SOURCES): src/switch_sai.thrift
	$(THRIFT) -o $(SRC) --gen py -r $(SRC)/switch_sai.thrift 

$(SAI_PY_HEADERS): $(SAI_HEADERS)
	$(CTYPESGEN) -I/usr/include -I$(SAI_HEADER_DIR) --include /usr/include/linux/limits.h $^ -o $@

clientlib: $(PY_SOURCES) $(SAI_PY_HEADERS)
	python setup.py sdist

$(ODIR)/%.o: src/gen-cpp/%.cpp
	$(CXX) $(CFLAGS) -c $^ -o $@

$(ODIR)/switch_sai_rpc_server.o: src/switch_sai_rpc_server.cpp
	$(CXX) $(CFLAGS) -c $^ -o $@ $(CFLAGS) -I$(SRC)/gen-cpp

$(ODIR)/saiserver.o: src/saiserver.cpp
	$(CXX) $(CFLAGS) -c $^ -o $@ $(CFLAGS) $(CDEFS) -I$(SRC)/gen-cpp -I$(SRC)

$(ODIR)/librpcserver.a: $(ODIR)/switch_sai_rpc.o $(ODIR)/switch_sai_types.o $(ODIR)/switch_sai_constants.o $(ODIR)/switch_sai_rpc_server.o
	ar rcs $(ODIR)/librpcserver.a $(ODIR)/switch_sai_rpc.o $(ODIR)/switch_sai_types.o $(ODIR)/switch_sai_constants.o $(ODIR)/switch_sai_rpc_server.o

saiserver: $(ODIR)/saiserver.o $(ODIR)/librpcserver.a
	$(CXX) $(LDFLAGS) $(ODIR)/switch_sai_rpc_server.o $(ODIR)/saiserver.o -o $@ \
		   $(ODIR)/librpcserver.a $(LIBS)

install-lib: $(ODIR)/librpcserver.a 
	$(INSTALL) -D $(ODIR)/librpcserver.a $(DESTDIR)/usr/lib/librpcserver.a
	$(INSTALL) -D $(SRC)/switch_sai_rpc_server.h $(DESTDIR)/usr/include/switch_sai_rpc_server.h

install-pylib: $(PY_SOURCES) $(SAI_PY_HEADERS)
	python setup.py install --prefix $(DESTDIR)/usr/local$(prefix)

install: install-lib install-pylib

clean:
	rm -rf $(ODIR) $(SRC)/gen-cpp $(SRC)/gen-py saiserver dist
