#
# This is a project Makefile. It is assumed the directory this Makefile resides in is a
# project subdirectory.
#

PROJECT_PATH := $(abspath .)
PROJECT_NAME :=  $(notdir $(PROJECT_PATH))
PROJECT_BOARD := evb
export PROJECT_PATH PROJECT_BOARD
#CONFIG_TOOLPREFIX :=

BLECONTROLLER_LIBS := std m0s1 m0s1s m0s1p m16s1 m0s0sp
BLECONTROLLER_LIB_DEFAULT := std
BLECONTROLLER_LIB_PREFIX := blecontroller_702_

-include ./proj_config.mk

ifeq ($(origin BL60X_SDK_PATH), undefined)
BL60X_SDK_PATH_GUESS ?= $(shell pwd)
BL60X_SDK_PATH ?= $(BL60X_SDK_PATH_GUESS)/../..
$(info ****** Please SET BL70X_SDK_PATH ******)
$(info ****** Trying SDK PATH [$(BL60X_SDK_PATH)])
endif

INCLUDE_COMPONENTS += bl702 bl702_std newlibc bl702_rf hosal vfs yloop utils cli blog
INCLUDE_COMPONENTS += blmtd blfdt
INCLUDE_COMPONENTS += easyflash4 coredump
INCLUDE_COMPONENTS += lmac154

ifeq ($(CONFIG_USB_CDC), 1)
INCLUDE_COMPONENTS += bl702_usb_cdc
endif

ifeq ($(CONFIG_BT),1)
ifeq ($(CONFIG_BLECONTROLLER_LIB),all)
	INCLUDE_COMPONENTS += $(addprefix $(BLECONTROLLER_LIB_PREFIX), $(BLECONTROLLER_LIBS))
else
	ifeq ($(findstring $(CONFIG_BLECONTROLLER_LIB), $(BLECONTROLLER_LIBS)),)
		INCLUDE_COMPONENTS += $(addprefix $(BLECONTROLLER_LIB_PREFIX), $(BLECONTROLLER_LIB_DEFAULT))
	else
		INCLUDE_COMPONENTS += $(addprefix $(BLECONTROLLER_LIB_PREFIX), $(CONFIG_BLECONTROLLER_LIB))
	endif
endif
INCLUDE_COMPONENTS += blestack
endif

ifeq ($(CONFIG_GEN_ROM),1)
    INCLUDE_COMPONENTS += bl702_freertos_rom
else
    ifeq ($(CONFIG_BUILD_ROM_CODE),1)
        INCLUDE_COMPONENTS += bl702_freertos
    else
        INCLUDE_COMPONENTS += bl702_freertos_rom
    endif
endif

ifeq ($(CONFIG_BT),1)
INCLUDE_COMPONENTS += blestack blecontroller
endif

INCLUDE_COMPONENTS += openthread_port openthread
INCLUDE_COMPONENTS += $(PROJECT_NAME)

# COMPONENT_COMMON := fs libc stage sys utils easyflash4
# COMPONENT_PLATFORM := soc/bl702 hosal
# COMPONENT_NETWORK := ble lmac154 lwip lwip_dhcpd netutils thread/openthread thread/openthread_port
# COMPONENT_DIRS := $(PROJECT_PATH)/$(PROJECT_NAME)
# COMPONENT_DIRS += $(addprefix $(BL60X_SDK_PATH)/components/, $(COMPONENT_COMMON))
# COMPONENT_DIRS += $(addprefix $(BL60X_SDK_PATH)/components/platform/, $(COMPONENT_PLATFORM))
# COMPONENT_DIRS += $(addprefix $(BL60X_SDK_PATH)/components/network/, $(COMPONENT_NETWORK))

CPPFLAGS += -DOT_FREERTOS_ENABLE=1
CPPFLAGS += -DOPENTHREAD_CONFIG_PLATFORM_INFO=\"BL702\"
# CPPFLAGS += -DOPENTHREAD_CONFIG_UART_CLI_RAW=1

CPPFLAGS += -DOPENTHREAD_CONFIG_HEAP_EXTERNAL_ENABLE=1

#CPPFLAGS += -DOPENTHREAD_CONFIG_LOG_MAC=1
#CPPFLAGS += -DOPENTHREAD_CONFIG_LOG_LEVEL=OT_LOG_LEVEL_DEBG
CPPFLAGS += -DOPENTHREAD_CONFIG_LOG_SUFFIX=\"\\r\\n\"

OPENTHREAD_RADIO:=0
ifeq ($(origin CONFIG_NCP), undefined)
    CPPFLAGS += -DOPENTHREAD_RADIO=0
    ifeq ($(CONFIG_FTD), 1)
        CPPFLAGS += -DOPENTHREAD_FTD=1
        CPPFLAGS += -DOPENTHREAD_MTD=0
    else
        CPPFLAGS += -DOPENTHREAD_FTD=0
        CPPFLAGS += -DOPENTHREAD_MTD=1

        CPPFLAGS += -DCONFIG_PP=${CONFIG_PP}
    endif

    CPPFLAGS += -DCONFIG_PREFIX=${CONFIG_PREFIX}

    ifeq ($(origin CONFIG_OTDEMO), undefined)
        #only cli command image
        ifeq ($(CONFIG_FTD), 1)
            # only build image with joiner and commissioner configuration for FTD device
            CPPFLAGS += -DOPENTHREAD_CONFIG_COMMISSIONER_ENABLE=1
            CPPFLAGS += -DOPENTHREAD_CONFIG_JOINER_ENABLE=1
        else
            # only build image with joiner configuration for MTD device
            CPPFLAGS += -DOPENTHREAD_CONFIG_JOINER_ENABLE=1
        endif
    else
        CPPFLAGS += -DCONFIG_OTDEMO=${CONFIG_OTDEMO}
        ifeq (${CONFIG_OTDEMO}, 2)
            # coap demo image
            CPPFLAGS += -DOPENTHREAD_CONFIG_COAP_API_ENABLE=1
        endif
    endif
else
    CPPFLAGS += -DDISABLE_PRINT
    CPPFLAGS += -DCONFIG_NCP=${CONFIG_NCP}
    CPPFLAGS += -DOPENTHREAD_CONFIG_NCP_HDLC_ENABLE=1
    CPPFLAGS += -DOPENTHREAD_ENABLE_NCP_SPINEL_ENCRYPTER=0

    ifeq ($(CONFIG_NCP), 0)
        OPENTHREAD_RADIO:=1
        CPPFLAGS += -DOPENTHREAD_RADIO=1
        CPPFLAGS += -DOPENTHREAD_FTD=0
        CPPFLAGS += -DOPENTHREAD_MTD=0
        CPPFLAGS += -DOPENTHREAD_SPINEL_CONFIG_OPENTHREAD_MESSAGE_ENABLE=0
    else
        CPPFLAGS += -DOPENTHREAD_RADIO=0
        ifeq ($(CONFIG_FTD), 1)
            CPPFLAGS += -DOPENTHREAD_FTD=1
            CPPFLAGS += -DOPENTHREAD_MTD=0
        else
            CPPFLAGS += -DOPENTHREAD_FTD=0
            CPPFLAGS += -DOPENTHREAD_MTD=1
        endif
        CPPFLAGS += -DOPENTHREAD_SPINEL_CONFIG_OPENTHREAD_MESSAGE_ENABLE=1
    endif
endif
export OPENTHREAD_RADIO

CPPFLAGS += -DOPENTHREAD_CONFIG_MLR_ENABLE=1
CPPFLAGS += -DOPENTHREAD_CONFIG_PING_SENDER_ENABLE=1
CPPFLAGS += -DOPENTHREAD_CONFIG_SNTP_CLIENT_ENABLE=1

CPPFLAGS += -DOPENTHREAD_CONFIG_TMF_NETDATA_SERVICE_ENABLE=0
CPPFLAGS += -DOPENTHREAD_CONFIG_SRP_SERVER_ENABLE=0

CPPFLAGS += -DOPENTHREAD_CONFIG_BORDER_ROUTER_ENABLE=0
CPPFLAGS += -DOPENTHREAD_CONFIG_BACKBONE_ROUTER_ENABLE=0

#CPPFLAGS += -DOPENTHREAD_CONFIG_PLATFORM_FLASH_API_ENABLE=1
CPPFLAGS += -DOPENTHREAD_CONFIG_PLATFORM_XTAL_ACCURACY=40

CPPFLAGS += -DOPENTHREAD_CONFIG_FILE=\"include/openthread-core-bl-config.h\"
CPPFLAGS += -DOPENTHREAD_PROJECT_CORE_CONFIG_FILE=\"include/openthread-core-bl-config.h\"
CPPFLAGS += -DOPENTHREAD_CORE_CONFIG_PLATFORM_CHECK_FILE=\"include/openthread-core-bl-config-check.h\"

CPPFLAGS += -DCONFIG_OT_ASSERT

CPPFLAGS += -DOT_FREERTOS_ENABLE=1
CPPFLAGS += -DCFG_CPP_ENABLE=1

include $(BL60X_SDK_PATH)/make_scripts_riscv/project.mk

EXTRA_LDFLAGS += -Wl,--print-memory-usage
ifdef CONFIG_CACHE_SIZE
EXTRA_LDFLAGS += -Wl,--defsym=__CACHE_SIZE=$(CONFIG_CACHE_SIZE)
endif


