## Set common environment variables
TOP ?= $(shell git rev-parse --show-toplevel)

include $(TOP)/Makefile.common

SYN_PATH    := $(BP_ME_DIR)/syn
TB_PATH     := $(BP_ME_DIR)/test/tb
MEM_PATH    := $(BP_COMMON_DIR)/test/mem

LOG_PATH    := $(BP_ME_DIR)/syn/logs
REPORT_PATH := $(BP_ME_DIR)/syn/reports
RESULTS_PATH := $(BP_ME_DIR)/syn/results

## End specific flags and variables

TB          ?= bp_cce
CFG         ?= e_bp_half_core_cfg

PADDR_WIDTH ?= 40

NUM_INSTR_P ?= 16
SEED_P ?= 1

# Select CCE ROM based on CFG and Coherence Protocol
ifeq ($(CFG), e_bp_half_core_cfg)
  NUM_LCE_P=1
  N_WG=64
else ifeq ($(CFG), e_bp_single_core_cfg)
  NUM_LCE_P=2
  N_WG=64
else ifeq ($(CFG), e_bp_dual_core_cfg)
  NUM_LCE_P=4
  N_WG=32
else ifeq ($(CFG), e_bp_quad_core_cfg)
  NUM_LCE_P=8
  N_WG=16
else ifeq ($(CFG), e_bp_oct_core_cfg)
  NUM_LCE_P=16
  N_WG=8
endif

COH_PROTO   ?= mesi
CCE_MEM_PATH = $(BP_ME_DIR)/src/asm/roms/$(COH_PROTO)
CCE_MEM      = bp_cce_inst_rom_$(COH_PROTO).mem

NUMS = $(shell seq 0 `expr $(NUM_LCE_P) - 1`)
BASE = bsg_trace_rom_
TRACE_ROMS = $(addsuffix .v, $(addprefix $(BASE), $(NUMS)))

DRAMSIM_CH_CFG  ?= DDR2_micron_16M_8b_x8_sg3E.ini
DRAMSIM_SYS_CFG ?= system.ini

.EXPORT_ALL_VARIABLES:

include $(BP_COMMON_DIR)/syn/Makefile.common
include $(BP_COMMON_DIR)/syn/Makefile.dc
include $(BP_COMMON_DIR)/syn/Makefile.regress
include $(BP_COMMON_DIR)/syn/Makefile.verilator
include $(BP_COMMON_DIR)/syn/Makefile.vcs

axe: AXE_LOG ?= ./axe.log
axe:
	python $(BP_ME_DIR)/software/py/axe_trace_filter.py $(AXE_LOG) > trace.axe
	cat trace.axe
	axe check TSO trace.axe

bsg_trace_rom_%.tr:
	-python $(TB_PATH)/$(TB)/bsg_trace_rom.py -m $(PADDR_WIDTH) -n $(NUM_INSTR_P) -s $(SEED_P) -i $* -l $(NUM_LCE_P) > $(TB_PATH)/$(TB)/$@

bsg_trace_rom_%.v: bsg_trace_rom_%.tr
	-python $(BASEJUMP_STL_DIR)/bsg_mem/bsg_ascii_to_rom.py $(TB_PATH)/$(TB)/$< bsg_trace_rom_$* > $(TB_PATH)/$(TB)/$@

copy_roms.v:
	-@mv $(TB_PATH)/$(TB)/bsg_trace_rom*.v $(BUILD_DIR)/
	-@rm $(TB_PATH)/$(TB)/bsg_trace_rom*.tr
	-@echo $(BUILD_DIR)/bsg_trace_rom*.v >> $(BUILD_DIR)/flist.vcs

copy_roms.sc:
	-@mv $(TB_PATH)/$(TB)/bsg_trace_rom*.v $(BUILD_DIR)/
	-@rm $(TB_PATH)/$(TB)/bsg_trace_rom*.tr
	-@echo $(BUILD_DIR)/bsg_trace_rom*.v >> $(BUILD_DIR)/flist.vcs

pre-build.v: $(TRACE_ROMS)
build.v: copy_roms.v

pre-verilate.sc: $(TRACE_ROMS)
verilate.sc: copy_roms.sc

lint.me:
	@echo "ME lint not supported"

regress.me.sc: dirs.sc
	$(MAKE) -j 1 build.sc sim.sc TB=bp_cce NUM_INSTR_P=$(NUM_INSTR_P)
	$(MAKE) -j 1 build.sc sim.sc TB=bp_cce_uc
	$(MAKE) -j 1 build.sc sim.sc TB=bp_cce_fsm NUM_INSTR_P=$(NUM_INSTR_P)
	$(MAKE) -j 1 build.sc sim.sc TB=bp_cce_fsm_uc

#	$(MAKE) -j 1 build.sc sim.sc TB=bp_cce_alu
#	$(MAKE) -j 1 build.sc sim.sc TB=bp_cce_gad

regress.me.v: dirs.v
	$(MAKE) -j 1 build.v sim.v TB=bp_cce NUM_INSTR_P=$(NUM_INSTR_P)
	$(MAKE) -j 1 build.v sim.v TB=bp_cce_uc
	$(MAKE) -j 1 build.v sim.v TB=bp_cce_fsm NUM_INSTR_P=$(NUM_INSTR_P)
	$(MAKE) -j 1 build.v sim.v TB=bp_cce_fsm_uc

regress: regress.me
regress.me: lint.me regress.me.sc regress.me.v

clean: clean.me
clean.me:
	rm -f *.axe

