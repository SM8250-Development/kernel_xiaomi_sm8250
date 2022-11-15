# SPDX-License-Identifier: GPL-2.0
#
# kbuild file for firmware/
#

# There are three cases to care about:
# 1. Building kernel with CONFIG_FIRMWARE_IN_KERNEL=y -- $(fw-shipped-y) should
#    include the firmware files to include, according to .config
# 2. 'make modules_install', which will install firmware for modules, and
#    _also_ for the in-kernel drivers when CONFIG_FIRMWARE_IN_KERNEL=n
# 3. 'make firmware_install', which installs all firmware, unconditionally.

# For the former two cases we want $(fw-shipped-y) and $(fw-shipped-m) to be
# accurate. In the latter case it doesn't matter -- it'll use $(fw-shipped-all).
# But be aware that the config file might not be included at all.
fw-shipped-$(CONFIG_TOUCHSCREEN_ST_FTS_V521) += st_fts_j1.ftb st_fts_j2.ftb st_fts_j2_1.ftb st_fts_j2_2.ftb st_fts_j2_3.ftb st_fts_j1s.ftb st_fts_j2s.ftb st_fts_j2s_default.ftb
fw-shipped-$(CONFIG_INPUT_PRESS_NDT) += ndt_fw.bin
fw-shipped-$(CONFIG_TOUCHSCREEN_GOODIX_GTX8) += goodix_gt9886_cfg_J2.bin goodix_gt9886_fw_J2.bin
fw-shipped-$(CONFIG_TOUCHSCREEN_ST_FTS_V521_SPI) += st_fts_l3a.ftb
fw-shipped-$(CONFIG_TOUCHSCREEN_ST_FTS_V521_SPI) += st_fts_l10a.ftb
fw-shipped-$(CONFIG_TOUCHSCREEN_NT36xxx_HOSTDL_SPI) += novatek_nt36675_j3s_fw01.bin novatek_nt36675_j3s_mp01.bin novatek_ts_fw.bin novatek_ts_mp.bin
fw-shipped-$(CONFIG_TOUCHSCREEN_NT36523_HOSTDL_SPI) += novatek_ts_fw_k81.bin novatek_ts_mp_k81.bin novatek_nt36523_k81_fw01.bin novatek_nt36523_k81_mp01.bin novatek_nt36523_k81a_fw02.bin novatek_nt36523_k81a_mp02.bin novatek_nt36523_k81a_fw01.bin novatek_nt36523_k81a_mp01.bin
fw-shipped-$(CONFIG_TOUCHSCREEN_FOCALTECH) += focaltech_ts_fw.bin
fw-shipped-$(CONFIG_TOUCHSCREEN_HIMAX_CHIPSET) += Himax_firmware.bin

fw-shipped-all := $(fw-shipped-y) $(fw-shipped-m) $(fw-shipped-)

quiet_cmd_ihex  = IHEX    $@
      cmd_ihex  = $(OBJCOPY) -Iihex -Obinary $< $@

quiet_cmd_ihex2fw  = IHEX2FW $@
      cmd_ihex2fw  = $(objtree)/$(obj)/ihex2fw $< $@

quiet_cmd_h16tofw  = H16TOFW $@
      cmd_h16tofw  = $(objtree)/$(obj)/ihex2fw -w $< $@

$(patsubst %,$(obj)/%.gen.S, $(fw-shipped-y)): %: $(wordsize_deps)
	$(call cmd,fwbin,$(patsubst %.gen.S,%,$@))

$(patsubst %,$(obj)/%.gen.o, $(fw-shipped-y)): %.gen.o: %

# .ihex is used just as a simple way to hold binary files in a source tree
# where binaries are frowned upon. They are directly converted with objcopy.
$(obj)/%: $(obj)/%.ihex
	$(call cmd,ihex)

# Don't depend on ihex2fw if we're installing and it already exists.
# Putting it after | in the dependencies doesn't seem sufficient when
# we're installing after a cross-compile, because ihex2fw has dependencies
# on stuff like /usr/lib/gcc/ppc64-redhat-linux/4.3.0/include/stddef.h and 
# thus wants to be rebuilt. Which it can't be, if the prebuilt kernel tree
# is exported read-only for someone to run 'make install'.
ifeq ($(INSTALL):$(wildcard $(obj)/ihex2fw),install:$(obj)/ihex2fw)
ihex2fw_dep :=
else
ihex2fw_dep := $(obj)/ihex2fw
endif

# .HEX is also Intel HEX, but where the offset and length in each record
# is actually meaningful, because the firmware has to be loaded in a certain
# order rather than as a single binary blob. Thus, we convert them into our
# more compact binary representation of ihex records (<linux/ihex.h>)
$(obj)/%.fw: $(obj)/%.HEX $(ihex2fw_dep)
	$(call cmd,ihex2fw)

# .H16 is our own modified form of Intel HEX, with 16-bit length for records.
$(obj)/%.fw: $(obj)/%.H16 $(ihex2fw_dep)
	$(call cmd,h16tofw)

obj-$(CONFIG_FIRMWARE_IN_KERNEL) += $(patsubst %,%.gen.o, $(fw-shipped-y))

# Remove .S files and binaries created from ihex
# (during 'make clean' .config isn't included so they're all in $(fw-shipped-))
targets := $(fw-shipped-) $(patsubst $(obj)/%,%, \
				$(shell find $(obj) -name \*.gen.S 2>/dev/null))

# Without this, built-in.o won't be created when it's empty, and the
# final vmlinux link will fail.
obj- := dummy
