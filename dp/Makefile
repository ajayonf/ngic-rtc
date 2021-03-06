# SPDX-License-Identifier: Apache-2.0
# Copyright(c) 2017 Intel Corporation

ifeq ($(RTE_SDK),)
$(error "Please define RTE_SDK environment variable")
endif

MAKEFLAGS += -j

# Default target, can be overriden by command line or environment
RTE_TARGET ?= x86_64-native-linuxapp-gcc

include $(RTE_SDK)/mk/rte.vars.mk
include $(dir $(realpath $(firstword $(MAKEFILE_LIST))))/*.mk

#DIRS-y += pipeline
# binary name
APP = ngic_dataplane
# all source are stored in SRCS-y
SRCS-y := main.c\
	pkt_handler.c\
	cdr.c\
	master_cdr.c\
	session_cdr.c\
	config.c\
	init.c\
	dataplane.c\
	gtpu.c\
	ether.c\
	ipv4.c\
	util.c\
	acl.c\
	meter.c\
	adc_table.c\
	pcc_table.c\
	sess_table.c\
	commands.c\
	stats.c\
	extended_cdr.c\
	timer_stats.c\
	kni_pkt_handler.c\
	gtpu_echo.c\
	pipeline/epc_packet_framework.c\
	pipeline/epc_arp.c\
	pipeline/epc_spns_dns.c\
	../interface/interface.c\
	../cp_dp_api/vepc_cp_dp_api.c\
	../test/simu_cp/nsb/nsb_test_util.c\
	../test/simu_cp/simu_cp.c\
	../test/simu_cp/pkt_filtrs.c\
	../interface/ipc/dp_ipc_api.c\
	../interface/udp/vepc_udp.c\

CFLAGS += -I$(SRCDIR)/
CFLAGS += -I$(SRCDIR)/../interface
CFLAGS += -I$(SRCDIR)/../interface/ipc
CFLAGS += -I$(SRCDIR)/../interface/udp
CFLAGS += -I$(SRCDIR)/../interface/sdn
CFLAGS += -I$(SRCDIR)/../interface/zmq
CFLAGS += -I$(SRCDIR)/../cp_dp_api
CFLAGS += -I$(SRCDIR)/../test/simu_cp
CFLAGS += -I$(SRCDIR)/../test/simu_cp/nsb
CFLAGS += -I$(SRCDIR)/pipeline
CFLAGS += -I$(SRCDIR)/../cp
CFLAGS += -I$(SRCDIR)/../lib/libsponsdn
# TODO: This is a hack
# We should not be looking into private structures
# Handle this in a patch to DPDK if necessary
# Check release notes
CFLAGS += -I$(RTE_SDK)/lib/librte_acl/

# Mandatory CFLAGS, LDFLAGS- DO NOT MODIFY
# #############################################################
CFLAGS += -Wno-psabi # suppress "The ABI for passing parameters with 64-byte alignment has changed in GCC 4.6"

CFLAGS += -DLDB_DP	# should be included for dataplane.

CFLAGS += -DDP_BUILD

CFLAGS += -Werror
CFLAGS += -Wunused-variable

# un-comment below line to Expose declaration of tdestroy()#
CFLAGS_config.o := -D_GNU_SOURCE

# ngic-dp application security check CFLAGS
# #############################################################
SECURITY_FLAGS = -D_FORTIFY_SOURCE=2 -fasynchronous-unwind-tables -fexceptions -fpie -fstack-protector-all -fstack-protector-strong -Wall -Werror=format-security -Werror=implicit-function-declaration -Wno-unused-function

CFLAGS += $(SECURITY_FLAGS)

# workaround for a gcc bug with noreturn attribute
#  http://gcc.gnu.org/bugzilla/show_bug.cgi?id=12603
ifeq ($(CONFIG_RTE_TOOLCHAIN_GCC),y)
CFLAGS_dataplane.o += -Wno-return-type
endif

#dpdk-18.02 change build directory structure
LDFLAGS += -L$(RTE_SRCDIR)/../lib/libsponsdn/x86_64-native-linuxapp-gcc/ -lsponsdn

LDFLAGS += -lhs -lhs_runtime -lstdc++ -lm -lcrypto

LDFLAGS += -lrte_pmd_af_packet

LDFLAGS += -lpcap

# ngic-dp application options CFLAGS
# #############################################################
# ASR- Shrink pipeline COREs for Run to Completion (rtc) model
ifneq (,$(findstring NGCORE_SHRINK, $(CFLAGS), $(EXTRA_CFLAGS)))
        SRCS-y += ddn.c
        SRCS-y += pipeline/epc_ul.c
        SRCS-y += pipeline/epc_dl.c
else
		SRCS-y += pipeline/epc_rx.c\
		pipeline/epc_tx.c\
		pipeline/epc_load_balance.c\
		pipeline/epc_worker.c\
		ddn_utils.c
endif

#validate both SDN_ODL_BUILD and ZMQ_COMM flag are not defined
ifneq (,$(findstring SDN_ODL_BUILD, $(CFLAGS), $(EXTRA_CFLAGS)))
ifneq (,$(findstring ZMQ_COMM, $(CFLAGS), $(EXTRA_CFLAGS)))
$(error "Both "SDN_ODL_BUILD" and "ZMQ_COMM" flags are defined. Exiting..")
endif
endif

ifneq (,$(findstring SDN_ODL_BUILD, $(CFLAGS), $(EXTRA_CFLAGS)))
	SRCS-y += ../interface/zmq/zmqsub.c
	SRCS-y += ../interface/zmq/zmqpub.c
	LDFLAGS += -L/usr/local/lib -lzmq
endif

ifeq ($(SGX_BUILD), 1)
	SRCS-y += ssl_client.c
	CFLAGS += -DSGX_CDR
	CFLAGS += -I$(NG_CORE)/linux-sgx/common/inc
	LDFLAGS += -lssl
endif

ifneq (,$(findstring ZMQ_COMM, $(CFLAGS), $(EXTRA_CFLAGS)))
	SRCS-y += ../interface/zmq/zmq_push_pull.c
	LDFLAGS += -L/usr/local/lib -lzmq
endif

# ngic-dp debug/testing/profiling options CFLAGS
# #############################################################
# gdb: un-comment 'CFLAGS += -g -O0' comment 'CFLAGS += -O3'
CFLAGS += -O3
#CFLAGS += -g -O0

# Un-comment to enable USE_AF_PACKET
#CFLAGS += -DUSE_AF_PACKET
ifneq (,$(findstring USE_AF_PACKET, $(CFLAGS), $(EXTRA_CFLAGS)))
	CFLAGS += -I/usr/include/libnl3
	LDFLAGS += -lmnl
endif

#un-comment below line to remove all log level for preformance testing.
CFLAGS += -DPERF_TEST

#un-comment below line to generate pcap on east-west interfaces
#CFLAGS += -DPCAP_GEN

# Un-comment below line to read fake cp config.
#CFLAGS += -DSIMU_CP

# Un-comment below line to send fake delete session request from simulator.
#Note: This will impact data traffic performance.
#CFLAGS += -DDEL_SESS_REQ

# VS- Used CP counters for maintain simulator stats.
ifneq (,$(findstring SIMU_CP, $(CFLAGS), $(EXTRA_CFLAGS)))
        SRCS-y += ../cp/cp_stats.c
endif

# Enable STATIC ARP for testing with il_nperf
# Un-comment below line to enable STATIC ARP
#CFLAGS += -DSTATIC_ARP

# Un-comment below line to print ADC, PCC, METER and SDF rule entry
# passed from FPC-SDN in add entry operation.
# Note : This flag works with Log level 'DEBUG'
#CFLAGS += -DPRINT_NEW_RULE_ENTRY

# Un-comment below line to get packet stats on each core.
CFLAGS += -DSTATS

# Un-comment below line to enable GTP-ECHO packets count on display(For SHRINK version).
#CFLAGS += -DEXSTATS

#un-comment below line to generate timer stats file.
#Note: This will impact data traffic performance.
#CFLAGS += -DTIMER_STATS

# Un-comment below line to enable AUTO-ANALYSIS mode for perforamance measurement
# Criteria to uncomment: if (PPS * test_duration) > 0.1M, then enable AUTO_ANALYSIS flag
#CFLAGS += -DAUTO_ANALYSIS

# ASR- Un-comment below line to enable PCC, ADC, CDR, FILTERING, METERING pipeline stages.
#CFLAGS += -DINCLUDE_PIPELINE_STAGE

# Un-comment below line to enable PCC, ADC, CDR, FILTERING, METERING pipeline stages.
#CFLAGS += -DINCLUDE_ADC_STAGE

# Un-comment below line if you have 16 x 1GB hugepages.
#CFLAGS += -DHUGE_PAGE_16GB

# ngic-dp inter-op/functional options CFLAGS
# #############################################################
# Un-comment below line to enable DDN(Downlink data notification)
#CFLAGS += -DDP_DDN

# Un-comment below line to enable Rating group CDRs.
# Disable/Comment out  RATING_GRP_CDR for performance profiling.
#CFLAGS += -DRATING_GRP_CDR

#un-comment below line to generate extended CDR file.
#Note: This will impact data traffic performance.
#CFLAGS += -DEXTENDED_CDR

# ASR- Un-comment below line to enable GTPU HEADER Sequence Number option.
#CFLAGS += -DGTPU_HDR_SEQNB

# Un-comment below line to enable hyperscan DPI.
# ASR- NGCORE_SHRINK for HYPERSCAN to be tested
#CFLAGS += -DHYPERSCAN_DPI

# Un-comment below line to enable PCC, ADC, CDR, FILTERING, METERING pipeline stages.
#CFLAGS += -DINCLUDE_MTR_STAGE

# ngic-dp  CFLAGS to be deprecated
# #############################################################
# Un-comment below line to enable DDN(Downlink data notification)
# Un-comment below line to get packet stats from command line.
#CFLAGS += -DCMDLINE_STATS

# Un-comment below line to enable core RX and core TX. All
# processing stages will be skipped.
#CFLAGS += -DRX_TX

# Un-comment below line to skip RX Meta access
#CFLAGS += -DSKIP_RX_META

# Un-comment below line to enable core RX, core LB and core TX. All
# processing stages will be skipped.
#CFLAGS += -DRX_LB_TX

# Un-comment below line to enable SDF Metering
#CFLAGS += -DSDF_MTR

# Un-comment below line to enable APN Metering
#CFLAGS += -DAPN_MTR

# Un-comment below line to enable ADC upfront.
#CFLAGS += -DADC_UPFRONT

# Un-comment below line to skip LB rte_hash_crc_4byte
# and enable LB based on UE ip last byte.
#CFLAGS += -DSKIP_LB_HASH_CRC

# set SGI port action handler equal to S1U action handler.
#CFLAGS += -DSKIP_LB_GTPU_AH

# Un-comment below line to read acl rules from file.
#CFLAGS += -DACL_READ_CFG

# Un-comment below line to configure DP Tables from DP app.
#CFLAGS += -DDP_TABLE_CONFIG

# ngic-dp include make overlays
# #############################################################
include $(RTE_SDK)/mk/rte.extapp.mk
