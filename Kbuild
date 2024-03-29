# We can build either as part of a standalone Kernel build or as
# an external module.  Determine which mechanism is being used
ifeq ($(MODNAME),)
	KERNEL_BUILD := 1
else
	KERNEL_BUILD := 0
endif

TFA_DEBUG                =1
TFADSP_DEBUG             =1
TFA_VERSION              =tfa9878
TFADSP_32BITS            =1
#TFADSP_DSP_MSG_PACKET_STRATEGY=1
TFADSP_DSP_BUFFER_POOL=1
TFA_EXCEPTION_AT_TRANSITION=1
TFA_USE_DEVICE_SPECIFIC_CONTROL=1
TFA_PROFILE_ON_DEVICE    =1
TFA_NO_SND_FORMAT_CHECK  =1
TFA_VOID_APIV_IN_FILE    =1
#TFA_SET_EXT_INTERNALLY   =1
TFA_MUTE_DURING_SWITCHING_PROFILE=1
TFA_SRC_DIR              =$(AUDIO_ROOT)/asoc/codecs/$(TFA_VERSION)

ifeq ($(KERNEL_BUILD), 1)
	# These are configurable via Kconfig for kernel-based builds
	# Need to explicitly configure for Android-based builds
	AUDIO_BLD_DIR := $(shell pwd)/kernel/msm-4.19
	AUDIO_ROOT := $(AUDIO_BLD_DIR)/techpack/audio
endif


ifeq ($(KERNEL_BUILD), 0)
	ifeq ($(CONFIG_ARCH_KONA), y)
		include $(AUDIO_ROOT)/config/konaauto.conf
		export
		INCS    +=  -include $(AUDIO_ROOT)/config/konaautoconf.h
	endif

endif

# As per target team, build is done as follows:
# Defconfig : build with default flags
# Slub      : defconfig  + CONFIG_SLUB_DEBUG := y +
#	      CONFIG_SLUB_DEBUG_ON := y + CONFIG_PAGE_POISONING := y
# Perf      : Using appropriate msmXXXX-perf_defconfig
#
# Shipment builds (user variants) should not have any debug feature
# enabled. This is identified using 'TARGET_BUILD_VARIANT'. Slub builds
# are identified using the CONFIG_SLUB_DEBUG_ON configuration. Since
# there is no other way to identify defconfig builds, QTI internal
# representation of perf builds (identified using the string 'perf'),
# is used to identify if the build is a slub or defconfig one. This
# way no critical debug feature will be enabled for perf and shipment
# builds. Other OEMs are also protected using the TARGET_BUILD_VARIANT
# config.

############ UAPI ############
UAPI_DIR :=	uapi
UAPI_INC :=	-I$(AUDIO_ROOT)/include/$(UAPI_DIR)

############ COMMON ############
COMMON_DIR :=	include
COMMON_INC :=	-I$(AUDIO_ROOT)/$(COMMON_DIR)
TFA9878_INC :=  -I$(TFA_SRC_DIR)/inc
############ MSM Soundwire ############

# for MSM Soundwire Codec
ifdef CONFIG_SND_SOC_TFA9878
	SND_SOC_TFA9878_OBJS += tfa98xx.o
	SND_SOC_TFA9878_OBJS += tfa_container.o
	SND_SOC_TFA9878_OBJS += tfa_dsp.o
	SND_SOC_TFA9878_OBJS += tfa_init.o
ifdef TFA_DEBUG
	SND_SOC_TFA9878_OBJS += tfa_debug.o
endif
endif

LINUX_INC +=	-Iinclude/linux

INCS +=		$(COMMON_INC) \
		$(UAPI_INC) \
		$(TFA9878_INC)

EXTRA_CFLAGS += $(INCS)


CDEFINES +=	-DANI_LITTLE_BYTE_ENDIAN \
		-DANI_LITTLE_BIT_ENDIAN \
		-DDOT11F_LITTLE_ENDIAN_HOST \
		-DANI_COMPILER_TYPE_GCC \
		-DANI_OS_TYPE_ANDROID=6 \
		-DPTT_SOCK_SVC_ENABLE \
		-Wall \
		-Werror \
		-D__linux__

CDEFINES += -DUSE_TFA9878 -DMTPLATFORM \
		-DTFA_DEBUG \
		-DTFADSP_32BITS \
		#-DTFADSP_DSP_MSG_PACKET_STRATEGY \
		-DTFADSP_DSP_BUFFER_POOL \
		-DTFADSP_DEBUG \
		-DTFA_USE_DEVICE_SPECIFIC_CONTROL \
		-DTFA_PROFILE_ON_DEVICE \
		-DTFA_NO_SND_FORMAT_CHECK \
		-DTFA_VOID_APIV_IN_FILE \
		#-DTFA_SET_EXT_INTERNALLY \
		-DTFA_MUTE_DURING_SWITCHING_PROFILE
			
KBUILD_CPPFLAGS += $(CDEFINES)

# Currently, for versions of gcc which support it, the kernel Makefile
# is disabling the maybe-uninitialized warning.  Re-enable it for the
# AUDIO driver.  Note that we must use EXTRA_CFLAGS here so that it
# will override the kernel settings.
ifeq ($(call cc-option-yn, -Wmaybe-uninitialized),y)
EXTRA_CFLAGS += -Wmaybe-uninitialized
endif
#EXTRA_CFLAGS += -Wmissing-prototypes

ifeq ($(call cc-option-yn, -Wheader-guard),y)
EXTRA_CFLAGS += -Wheader-guard
endif

ifeq ($(KERNEL_BUILD), 0)
# TO BE REVISED FOR OTHER PLATFORMS THAN Q
KBUILD_EXTRA_SYMBOLS +=$(OUT)/obj/vendor/qcom/opensource/audio-kernel/ipc/Module.symvers
KBUILD_EXTRA_SYMBOLS +=$(OUT)/obj/vendor/qcom/opensource/audio-kernel/dsp/Module.symvers
KBUILD_EXTRA_SYMBOLS +=$(OUT)/obj/vendor/qcom/opensource/audio-kernel/asoc/Module.symvers
KBUILD_EXTRA_SYMBOLS +=$(OUT)/obj/vendor/qcom/opensource/audio-kernel/asoc/codecs/Module.symvers
KBUILD_EXTRA_SYMBOLS +=$(OUT)/obj/vendor/qcom/opensource/audio-kernel/soc/Module.symvers
KBUILD_EXTRA_SYMBOLS +=$(OUT)/obj/vendor/qcom/opensource/audio-kernel/asoc/codecs/tfa9878/Module.symvers
endif

# Module information used by KBuild framework
obj-$(CONFIG_SND_SOC_TFA9878) += tfa9878_dlkm.o
tfa9878_dlkm-y := $(SND_SOC_TFA9878_OBJS)
# inject some build related information
DEFINES += -DBUILD_TIMESTAMP=\"$(shell date -u +'%Y-%m-%dT%H:%M:%SZ')\"
