# System-specific modifications may be necessary, especially to KOKKOS_PATH,
# which must be set to the Kokkos install directory or root of the source tree.
KOKKOS_PATH ?= $(HOME)/Kokkos/kokkos
KOKKOS_DEVICES ?= Hip
# These lines convert device names to Title Case for compatibility with old versions of Kokkos
override KOKKOS_DEVICES := $(shell tr ',' ' ' <<<"$(KOKKOS_DEVICES)")
override KOKKOS_DEVICES := $(foreach device,$(KOKKOS_DEVICES),$(shell dev=$(device);$\
	echo $$(tr '[:lower:]' '[:upper:]' <<<$${dev:0:1})$$(tr '[:upper:]' '[:lower:]' <<<$${dev:1})))
override KOKKOS_DEVICES := $(shell tr ' ' ',' <<<"$(KOKKOS_DEVICES)")
override KOKKOS_DEVICES := $(subst Openmp,OpenMP,$(KOKKOS_DEVICES))
EXE = mpi_heat_conduction
# Default build command/wrapper (Threads, Openmp)
CXX = mpicxx
LINK = $(CXX)
# Override build command/wrapper when using a GPU device
ifneq (,$(findstring Cuda,$(KOKKOS_DEVICES)))
CXX = $(KOKKOS_PATH)/bin/nvcc_wrapper -ccbin mpicxx
LINK = $(CXX)
endif
ifneq (,$(findstring Hip,$(KOKKOS_DEVICES)))
KOKKOS_ARCH ?= Vega90A
CXX = hipcc
LINK = $(CXX)
endif
default: all
SRC = $(EXE:=_solution.cpp)
OBJ = $(SRC:.cpp=.o)
include $(KOKKOS_PATH)/Makefile.kokkos
ifeq ($(CXX),hipcc)
LDFLAGS += -L${MPICH_DIR}/lib -lmpi ${CRAY_XPMEM_POST_LINK_OPTS} \
        -lxpmem ${PE_MPICH_GTL_DIR_amd_gfx90a} ${PE_MPICH_GTL_LIBS_amd_gfx90a}
CPPFLAGS += -I${MPICH_DIR}/include
endif
# The two steps below are deliberately separate to avoid a parallel make race between 
# Kokkos file generation and the compilation commands that consume the generated code
all: force
	@$(MAKE) $(KOKKOS_CONFIG_HEADER)
	@$(MAKE) $(EXE)
$(EXE): $(OBJ) $(KOKKOS_LINK_DEPENDS)
	$(LINK) $(LDFLAGS) $(KOKKOS_LDFLAGS) $< $(KOKKOS_LIBS) -o $@
clean: kokkos-clean 
	rm -f *.o $(EXE)
# Compilation rules
%.o:%.cpp $(KOKKOS_CPP_DEPENDS)
	$(CXX) $(CPPFLAGS) $(KOKKOS_CPPFLAGS) $(CXXFLAGS) $(KOKKOS_CXXFLAGS) -c $< -o $@
.PHONY: force clean

