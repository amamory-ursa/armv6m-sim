###############################################################################
# Inputs
###############################################################################
# SRC_DIR
SRC_DIR ?= .
# INCLUDE_PATH
# TARGET
TARGET ?= example
# EXTRA_CFLAGS
# EXTRA_LIBS
# EXTRA_LIBDIRS
# GCC_PREFIX     = arch-toolchain-
# OPT            = [0-2]
OPT ?= 0
# FPIC           = yes | no
# RUN_PREFIX
# RUN_ARGS
# ARCH
# ARCH_CFLAGS
# ARCH_LFLAGS
# COMPILER       = g++ | gcc

###############################################################################
# ARM-V6M Flags
###############################################################################
# GCC_PREFIX     = arch-toolchain-
GCC_PREFIX=arm-none-eabi-
# ARCH_CFLAGS
ARCH_CFLAGS=-mthumb -mcpu=cortex-m0 -ffunction-sections -fdata-sections -fmessage-length=0
# ARCH_LFLAGS
ARCH_LFLAGS=-nostartfiles -nodefaultlibs -lgcc -lm -mcpu=cortex-m0 -mthumb -Wl,--gc-sections
# FPIC           = yes | no
FPIC = no
# SHARED_LIB     = yes | no
SHARED_LIB = no
# COMPILER       = g++ | gcc
COMPILER = gcc

###############################################################################
# Board Settings
###############################################################################
ARCH_LFLAGS += -T./linker_script.ld

RUN_PREFIX ?= ../armv6m-sim -f


###############################################################################
# Variables
###############################################################################
ifneq ($(ARCH),)
  ARCH_TGT_DIR=$(ARCH)
else
  ARCH_TGT_DIR=linux
endif

OBJ_DIR      ?= obj.$(ARCH_TGT_DIR).$(TARGET)/
DEP_DIR      ?= dep.$(ARCH_TGT_DIR).$(TARGET)/
EXE_DIR      ?= build.$(ARCH_TGT_DIR).$(TARGET)/

###############################################################################
# Variables: GCC
###############################################################################
QUIET        ?= yes

GCC_PREFIX   ?= 
COMPILER     ?= g++

ifeq ($(QUIET),yes)
GCC          = @$(GCC_PREFIX)$(COMPILER)
OBJCOPY      = @$(GCC_PREFIX)objcopy
OBJDUMP      = @$(GCC_PREFIX)objdump
else
GCC          = $(GCC_PREFIX)$(COMPILER)
OBJCOPY      = $(GCC_PREFIX)objcopy
OBJDUMP      = $(GCC_PREFIX)objdump
endif

###############################################################################
# Variables: Compilation flags
###############################################################################

# Additional include directories
INCLUDE_PATH += $(SRC_DIR)

# Flags
CFLAGS       = $(ARCH_CFLAGS) -O$(OPT)
ifeq ($(FPIC), yes)
CFLAGS       += -fpic
endif
CFLAGS       += $(patsubst %,-I%,$(INCLUDE_PATH))
CFLAGS       += $(EXTRA_CFLAGS)

LFLAGS        = $(ARCH_LFLAGS)
LFLAGS       += $(patsubst %,-L%,$(EXTRA_LIBDIRS))
LFLAGS       += $(EXTRA_LIBS)

###############################################################################
# Variables: Lists of objects, source and deps
###############################################################################
# SRC / Object list
src2obj       = $(OBJ_DIR)$(patsubst %$(suffix $(1)),%.o,$(notdir $(1)))
src2dep       = $(DEP_DIR)$(patsubst %,%.d,$(notdir $(1)))

SRC          := $(EXTRA_SRC) $(foreach src,$(SRC_DIR),$(wildcard $(src)/*.cpp)) $(foreach src,$(SRC_DIR),$(wildcard $(src)/*.c))
OBJ          ?= $(foreach src,$(SRC),$(call src2obj,$(src)))
DEPS         ?= $(foreach src,$(SRC),$(call src2dep,$(src)))

###############################################################################
# Rules: Compilation macro
###############################################################################
# Dependancy generation
DEPFLAGS      = -MT $$@ -MMD -MP -MF $(call src2dep,$(1))

define template_c
$(call src2obj,$(1)): $(1) | $(OBJ_DIR) $(DEP_DIR)
	@echo "# Compiling $(notdir $(1))"
	$(GCC) $(CFLAGS) $(DEPFLAGS) -c $$< -o $$@
endef

###############################################################################
# Rules
###############################################################################
BUILD_TARGETS = $(EXE_DIR)$(TARGET)

ifeq ($(ENABLE_BIN),yes)
  BUILD_TARGETS += $(EXE_DIR)$(TARGET).bin
endif
ifeq ($(ENABLE_LST),yes)
  BUILD_TARGETS += $(EXE_DIR)$(TARGET).lst
endif

all: $(BUILD_TARGETS)

$(OBJ_DIR) $(DEP_DIR) $(EXE_DIR):
	@mkdir -p $@

$(foreach src,$(SRC),$(eval $(call template_c,$(src))))

$(EXE_DIR)$(TARGET): $(OBJ) | $(EXE_DIR) 
	@echo "# Building $(notdir $@)"
	$(GCC) -o $(EXE_DIR)$(TARGET) $(OBJ) $(LFLAGS)

$(EXE_DIR)$(TARGET).bin: $(EXE_DIR)$(TARGET)
	@echo "# Building $(notdir $@)"
	$(OBJCOPY) -O binary $< $@

$(EXE_DIR)$(TARGET).lst: $(EXE_DIR)$(TARGET)
	@echo "# Building $(notdir $@)"
	$(OBJDUMP) -d $< > $@

run:
	$(RUN_PREFIX) $(EXE_DIR)$(TARGET) $(RUN_ARGS)

clean:
	rm -rf $(EXE_DIR) $(OBJ_DIR) $(DEP_DIR)

###############################################################################
# Rules: Dependancies
###############################################################################
EXCLUDE_DEPS := clean
ifeq (0, $(words $(findstring $(MAKECMDGOALS), $(EXCLUDE_DEPS))))
-include $(DEPS)
endif
