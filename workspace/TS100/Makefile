
ifndef model
model:=TS100
endif
OUTPUT_EXE=$(model)_$(lang)
ifndef lang
lang:= EN
endif

# Enumerate all of the include directories
APP_INC_DIR = ./Core/Inc
INC_CMSIS_DEVICE = ./Core/BSP/Miniware/Vendor/CMSIS/Device/ST/STM32F1xx/Include
CMSIS_CORE_INC_DIR = ./Core/BSP/Miniware/Vendor/CMSIS/Include
HAL_INC_DIR = ./Core/BSP/Miniware/Vendor/STM32F1xx_HAL_Driver/Inc
HAL_LEGACY_INC_DIR = ./Core/BSP/Miniware/Vendor/STM32F1xx_HAL_Driver/Inc/Legacy
FRTOS_CMIS_INC_DIR = ./Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS
FRTOS_INC_DIR = ./Middlewares/Third_Party/FreeRTOS/Source/include
MINIWARE_STARTUP_DIR = ./Startup
DRIVER_INC_DIR =./Core/Drivers
BSP_INC_DIR = ./Core/BSP
THREADS_INC_DIR = ./Core/Threads
MINIWARE_INC_DIR = ./Core/BSP/Miniware
PINE_INC_DIR = ./Core/BSP/Pine64
PINE_VENDOR_INC_DIR = ./Core/BSP/Pine64/Vendor/Lib
PINE_RISCV_INC_DIR = ./Core/BSP/Pine64/Vendor/RISCV
PINE_N200_INC_DIR = ./Core/BSP/Pine64/N200
SOURCE_THREADS_DIR = ./Core/Threads
SOURCE_CORE_DIR = ./Core/Src
SOURCE_DRIVERS_DIR = ./Core/Drivers
INC_PD_DRIVERS_DIR = ./Core/Drivers/FUSB302
SOURCE_MIDDLEWARES_DIR = ./Middlewares

# Device dependent settings
ifeq ($(model),$(filter $(model),TS100 TS80 TS80P))
$(info Building for Miniware )
DEVICE_INCLUDES = -I$(MINIWARE_INC_DIR) \
	-I$(INC_CMSIS_DEVICE)\
	-I$(CMSIS_CORE_INC_DIR)	\
	-I$(HAL_INC_DIR)	\
	-I$(HAL_LEGACY_INC_DIR)	\
	-I$(INC_PD_DRIVERS_DIR)
DEVICE_BSP_DIR = ./Core/BSP/Miniware
S_SRCS := $(shell find $(MINIWARE_STARTUP_DIR) -type f -name '*.S')
LDSCRIPT=stm32f103.ld
DEV_GLOBAL_DEFS= -D STM32F103T8Ux -D STM32F1 -D STM32 -D USE_HAL_DRIVER -D STM32F103xB -D USE_RTOS_SYSTICK -D GCC_ARMCM3		\
		-D ARM_MATH_CM3 	\
		-D STM32F10X_MD 
DEV_LDFLAGS=-lm -Wl,--gc-sections -flto
DEV_AFLAGS=
DEV_CFLAGS=-flto -D GCC_ARMCM3		\
		-D ARM_MATH_CM3 	\
		-D STM32F10X_MD		
DEV_CXXFLAGS=-flto -D GCC_ARMCM3		\
		-D ARM_MATH_CM3 	\
		-D STM32F10X_MD		
CPUFLAGS= -mcpu=cortex-m3     \
		  -mthumb 			\
		  -mfloat-abi=soft
flash_size=64k
bootldr_size=0x4000
endif

ifeq ($(model),Pinecil)
$(info Building for Pine64 )
DEVICE_INCLUDES = -I$(PINE_INC_DIR) \
-I$(PINE_INC_DIR)/N200 \
	-I$(PINE_VENDOR_INC_DIR) \
	-I$(PINE_RISCV_INC_DIR) \
	-I$(PINE_N200_INC_DIR)\
	-I$(INC_PD_DRIVERS_DIR)
	
DEVICE_BSP_DIR = ./Core/BSP/Pine64
S_SRCS := ./Core/BSP/Pine64/N200/portasm.S  
$(info $(S_SRCS) )
ASM_INC = -I$(PINE_RISCV_INC_DIR)
LDSCRIPT=GD32VF103xB.ld
flash_size=128k
bootldr_size=0x0
# Flags
CPUFLAGS= -march=rv32imac \
			-mabi=ilp32 \
			-mtune=size  \
			-mcmodel=medlow  \
			-DGD32VF103C_START \
			-DUSE_STDPERIPH_DRIVER 
DEV_LDFLAGS= -nostartfiles -Xlinker --gc-sections
DEV_AFLAGS= -nostartfiles -ffreestanding
DEV_GLOBAL_DEFS=-msmall-data-limit=8 -mno-save-restore -Os -fmessage-length=0 -fabi-version=11 
DEV_CFLAGS= -MMD -MP -MF"$(@:%.o=%.d)" -MT"$(@:%.o=%.d)" 
DEV_CXXFLAGS= -MMD -MP -MF"$(@:%.o=%.d)" -MT"$(@:%.o=%.d)" 
endif

INCLUDES = -I$(APP_INC_DIR)	\
	-I$(FRTOS_CMIS_INC_DIR)	\
	-I$(FRTOS_INC_DIR)	\
	-I$(DRIVER_INC_DIR) \
	-I$(BSP_INC_DIR) \
	-I$(THREADS_INC_DIR) \
	  $(DEVICE_INCLUDES) 
	  
SOURCE := $(shell find $(SOURCE_THREADS_DIR) -type f -name '*.c') \
$(shell find $(SOURCE_CORE_DIR) -type f -name '*.c') \
$(shell find $(SOURCE_DRIVERS_DIR) -type f -name '*.c') \
$(shell find $(DEVICE_BSP_DIR) -type f -name '*.c') \
$(shell find $(SOURCE_MIDDLEWARES_DIR) -type f -name '*.c') 
SOURCE_CPP := $(shell find $(SOURCE_THREADS_DIR) -type f -name '*.cpp') \
$(shell find $(SOURCE_CORE_DIR) -type f -name '*.cpp') \
$(shell find $(SOURCE_DRIVERS_DIR) -type f -name '*.cpp') \
$(shell find $(DEVICE_BSP_DIR) -type f -name '*.cpp') \
$(shell find $(SOURCE_MIDDLEWARES_DIR) -type f -name '*.cpp') 
# output folder
HEXFILE_DIR=Hexfile

# temporary objects folder
OUTPUT_DIR_BASE=Objects
OUTPUT_DIR=Objects/$(model)
# code optimisation ------------------------------------------------------------
OPTIM=-Os  -finline-small-functions -findirect-inlining -fdiagnostics-color -ffunction-sections -fdata-sections


# global defines ---------------------------------------------------------------
GLOBAL_DEFINES += $(DEV_GLOBAL_DEFS) -D USE_RTOS_SYSTICK -D LANG_$(lang) -D LANG -D MODEL_$(model) -D VECT_TAB_OFFSET=$(bootldr_size)U

ifdef swd_enable
	GLOBAL_DEFINES += -DSWD_ENABLE
endif

# Enable debug code generation
DEBUG=-g3
# Without debug code
#DEBUG=


# libs -------------------------------------------------------------------------
LIBS=

# ------------------------------------------------------------------------------
COMPILER=gcc
# arm-none-eabi is the general ARM compiler, 
#riscv-none-embed- is the riscv compiler
ifeq ($(model),$(filter $(model),TS100 TS80 TS80P))
COMPILER_PREFIX=arm-none-eabi
endif
ifeq ($(model),Pinecil)
COMPILER_PREFIX=riscv-nuclei-elf
endif
# programs ---------------------------------------------------------------------
CC=$(COMPILER_PREFIX)-gcc
CPP=$(COMPILER_PREFIX)-g++
AS=$(COMPILER_PREFIX)-as
OBJCOPY=$(COMPILER_PREFIX)-objcopy
OBJDUMP=$(COMPILER_PREFIX)-objdump
SIZE=$(COMPILER_PREFIX)-size
SREC=srec_cat
SREC_INFO=srec_info

# For the Pinecil, use GCC rather than as? (Copying other makefiles)
ifeq ($(model),Pinecil)
AS=$(COMPILER_PREFIX)-gcc -x assembler-with-cpp
endif

# linker flags -----------------------------------------------------------------
LINKER_FLAGS=   -Wl,--gc-sections 		                      \
                -Wl,--wrap=malloc                             \
			    -Wl,--wrap=free                               \
			    -o$(OUT_HEXFILE).elf 	                      \
			    -Wl,-Map=$(OUT_HEXFILE).map                   \
			    -lm                                           \
			    -Wl,--undefined=vTaskSwitchContext            \
			    -Wl,--undefined=pxCurrentTCB                  \
			    -Wl,--defsym=__FLASH_SIZE__=$(flash_size)     \
			    -Wl,--defsym=__BOOTLDR_SIZE__=$(bootldr_size) \
				$(DEV_LDFLAGS)                                \
			    --specs=nano.specs

# compiler flags ---------------------------------------------------------------


CHECKOPTIONS=		-Wall 				\
			-Wextra				\
			-Wunused			\
			-Wcomment 			\
			-Wtrigraphs			\
			-Wuninitialized		\
			-Wmissing-braces	\
			-Wfloat-equal		\
			-Wunreachable-code	\
			-Wswitch-default	\
			-Wreturn-type		\
			-Wundef				\
			-Wparentheses		\
			-Wnonnull			\
			-Winit-self			\
			-Wmissing-include-dirs	\
			-Wsequence-point	\
			-Wswitch			\
			-Wformat			\
			-Wsign-compare		\
			-Waddress			\
			-Waggregate-return	\
			-Wmissing-field-initializers	\
			-Winline			\
			-Wshadow			\
			-Wno-unused-parameter \
			-Wdouble-promotion \
			-Werror 
			


CHECKOPTIONS_C=		-Wbad-function-cast


CXXFLAGS=$(DEV_CXXFLAGS) \
        $(CPUFLAGS) \
		$(DEBUG) 			\
		$(INCLUDES) \
		$(GLOBAL_DEFINES) 	\
		-D${COMPILER}  		\
		-MMD 				\
		$(CHECKOPTIONS)		\
		-std=c++11			\
		$(OPTIM) 			\
		-fno-common			\
		-ffreestanding		\
		-fno-rtti 			\
		-fno-exceptions 	\
		-fno-non-call-exceptions \
		-fno-use-cxa-atexit		\
		-fno-strict-aliasing 		\
		-fno-rtti 			\
		-fno-exceptions		\
		-fno-threadsafe-statics \
		-T$(LDSCRIPT)


CFLAGS=$(DEV_CFLAGS) \
        $(CPUFLAGS) \
		$(DEBUG) 			\
		$(INCLUDES) 			\
		$(CHECKOPTIONS_C) 		\
		$(GLOBAL_DEFINES) 		\
		-D${COMPILER}  			\
		-MMD 				\
		-std=gnu99			\
		$(OPTIM) 			\
		-fno-common			\
		-ffreestanding			\
		-T$(LDSCRIPT)			\
		-c                      
		
		
AFLAGS= $(CPUFLAGS)   \
		$(DEV_AFLAGS) \
		$(DEBUG)      \
		$(ASM_INC)    \
		$(INCLUDES)


CFLAGS += -ffunction-sections -fdata-sections
CXXFLAGS += -ffunction-sections -fdata-sections



OBJS = $(SOURCE:.c=.o)
OBJS_CPP = $(SOURCE_CPP:.cpp=.o)
OBJS_S = $(S_SRCS:.S=.o)



OUT_OBJS=$(addprefix $(OUTPUT_DIR)/,$(OBJS))
OUT_OBJS_CPP=$(addprefix $(OUTPUT_DIR)/,$(OBJS_CPP))
OUT_OBJS_S=$(addprefix $(OUTPUT_DIR)/,$(OBJS_S))
OUT_HEXFILE=$(addprefix $(HEXFILE_DIR)/,$(OUTPUT_EXE))

all: $(OUT_HEXFILE).hex $(OUT_HEXFILE).bin

#
# The rule to create the target directory
#

# Create hexfile
%.hex : %.elf
	$(SIZE) $^
	$(OBJCOPY) $^ -O ihex $@

%.bin : %.elf
	$(SIZE) $^
	$(OBJCOPY) $^ -O binary $@

$(OUT_HEXFILE).elf : $(OUT_OBJS) $(OUT_OBJS_CPP) $(OUT_OBJS_S) Makefile $(LDSCRIPT)
	@test -d $(@D) || mkdir -p $(@D)
	@echo Linking $(OUTPUT_EXE).elf
	@echo $(LINKER_FLAGS)
	@$(CPP) $(CXXFLAGS) $(OUT_OBJS_S) $(OUT_OBJS) $(OUT_OBJS_CPP) $(LIBS) $(LINKER_FLAGS)

$(OUT_OBJS): $(OUTPUT_DIR)/%.o : %.c Makefile
	@test -d $(@D) || mkdir -p $(@D)
	@echo Compiling ${<}
	@$(CC) -c $(CFLAGS) $< -o $@
	@$(OBJDUMP) -d -S $@ > $@.lst

$(OUT_OBJS_CPP): $(OUTPUT_DIR)/%.o : %.cpp Makefile
	@test -d $(@D) || mkdir -p $(@D)
	@echo Compiling ${<}
	@$(CPP) -c $(CXXFLAGS) $< -o $@
	@$(OBJDUMP) -d -S $@ > $@.lst
	
$(OUT_OBJS_S): $(OUTPUT_DIR)/%.o: %.S Makefile
	@test -d $(@D) || mkdir -p $(@D)
	@echo 'Building file: $<'
	@echo 'Invoking: MCU GCC Assembler'
	@echo @$(AS) -c $(AFLAGS) $< -o $@
	@$(AS) -c $(AFLAGS) $< -o $@
	@echo 'Finished building: $<'
	@echo ' '


clean :
	rm -Rf $(OUTPUT_DIR_BASE)
	rm -Rf $(HEXFILE_DIR)

# pull in dependency info for *existing* .o files
-include $(OUT_OBJS:.o=.d)
-include $(OUT_OBJS_CPP:.o=.d)

