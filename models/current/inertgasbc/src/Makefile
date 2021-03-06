# Set the environment variable TMMROOT to point to the top level of the TMM code
#
TMMBASE = ${TMMROOT}/driver/current
MODELBASE = ${TMMROOT}/models/current/inertgasbc/src

include ${PETSC_DIR}/lib/petsc/conf/variables
include ${PETSC_DIR}/lib/petsc/conf/rules

# Set compiler-dependent flags
FPPFLAG=
ifeq ($(strip $(MPIFC_SHOW)),)
	COMPILER=$(FC)
else
	COMPILER=$(word 1,$(MPIFC_SHOW))
endif

ifneq (,$(findstring ifort,$(COMPILER)))
	FC_FLAGS+=
endif
ifneq (,$(findstring gfortran,$(COMPILER)))
	FC_FLAGS+=
endif
ifneq (,$(findstring xlf,$(COMPILER)))
	FPPFLAG+=-WF,
	FC_FLAGS+=-qzerosize
endif
ifneq (,$(findstring ftn,$(COMPILER)))
	FC_FLAGS+=
endif

MYPATHS = $(TMMBASE):$(MODELBASE)

INCLUDEPATHS = $(addprefix -I,$(subst :, ,$(MYPATHS)))

VPATH = $(MYPATHS)

EXE = tmminertgasbc

SRCTMM = tmm_main.c tmm_forward_step.c tmm_write.c tmm_forcing_utils.c \
        tmm_profile_utils.c tmm_timer.c petsc_signal_utils.c petsc_matvec_utils.c

SRCMODEL = tmm_external_forcing.c tmm_monitor.c tmm_misfit.c external_bc_inert_gas.c
SRFMODEL = inert_gas_bc.F inert_gas_chem.F inert_gas_diagnostics.F seawater.F

OPTFILE =

ifneq ($(strip $(OPTFILE)),)
# Make sure OPTFILE is present in the current directory
  ifeq (,$(wildcard $(OPTFILE)))
    $(error $(OPTFILE) does not exist in the local directory!)
  endif
  CPPFLAGS+=$(shell awk '/^\#define/ {print "-D"$$2}' $(OPTFILE))
  FPPFLAGS+=$(shell awk '/^\#define/ {print "$(FPPFLAG)-D"$$2}' $(OPTFILE))
endif

OBJTMM = $(SRCTMM:.c=.o)
OBJCMODEL = $(SRCMODEL:.c=.o)
OBJFMODEL = $(SRF90MODEL:.F90=.o) $(SRFMODEL:.F=.o)

$(OBJCMODEL) $(OBJFMODEL): $(OPTFILE)

%.o %.mod: %.F90
	$(FC) $(INCLUDEPATHS) $(FC_FLAGS) $(FFLAGS) $(FCPPFLAGS) -c -o $$(echo $@ | sed -e 's/mod$$/o/') $(FC_MODULE_OUTPUT_FLAG) $(dir $@) $<

%.o: %.F
	$(FC) $(INCLUDEPATHS) $(FC_FLAGS) $(FFLAGS) $(FCPPFLAGS) -c -o $@ $<

%.o: %.c
	$(CC) $(INCLUDEPATHS) $(PCC_FLAGS) $(CFLAGS) $(CCPPFLAGS) -c -o $@ $<

$(EXE): $(OBJTMM) $(OBJCMODEL) $(OBJFMODEL)
	-$(CLINKER) -o $@ $(OBJTMM) $(OBJCMODEL) $(OBJFMODEL) $(PETSC_MAT_LIB)

cleanall:
	make clean
	rm -f *.mod *.i *.i90
