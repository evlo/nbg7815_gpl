#
# Copyright (C) 2007-2016 OpenWrt.org
# Copyright (C) 2010 Vertical Communications
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/version.mk

PKG_NAME:=base-files
PKG_RELEASE:=157.2

PKG_FILE_DEPENDS:=$(PLATFORM_DIR)/ $(GENERIC_PLATFORM_DIR)/base-files/
PKG_BUILD_DEPENDS:=opkg/host usign/host
PKG_LICENSE:=GPL-2.0

PKG_CONFIG_DEPENDS := CONFIG_SIGNED_PACKAGES

include $(INCLUDE_DIR)/package.mk

PRODUCT_NAME=$(CONFIG_ZYXEL_PROJECT_NAME)

ifneq ($(DUMP),1)
  TARGET:=-$(BOARD)
  ifneq ($(wildcard $(PLATFORM_DIR)/base-files-$(PROFILE) $(PLATFORM_SUBDIR)/base-files-$(PROFILE)),)
    TARGET:=$(TARGET)-$(PROFILE)
  endif
endif

define Package/base-files
  SECTION:=base
  CATEGORY:=Base system
  DEPENDS:=+netifd +libc +procd +jsonfilter +SIGNED_PACKAGES:usign
  TITLE:=Base filesystem for OpenWrt
  URL:=http://openwrt.org/
  VERSION:=$(PKG_RELEASE)-$(REVISION)
endef

define Package/base-files/conffiles
/etc/config/network
/etc/config/system
/etc/hosts
/etc/inittab
/etc/group
/etc/passwd
/etc/shadow
/etc/profile
/etc/protocols
/etc/services
/etc/shells
/etc/sysctl.conf
/etc/rc.local
/etc/sysupgrade.conf
/etc/config/
/etc/dropbear/
/etc/crontabs/
$(call $(TARGET)/conffiles)
endef

define Package/base-files/description
 This package contains a base filesystem and system scripts for OpenWrt.
endef

ifneq ($(CONFIG_PREINITOPT),)
define ImageConfigOptions
	mkdir -p $(1)/lib/preinit
	echo 'pi_suppress_stderr="$(CONFIG_TARGET_PREINIT_SUPPRESS_STDERR)"' >$(1)/lib/preinit/00_preinit.conf
	echo 'fs_failsafe_wait_timeout=$(if $(CONFIG_TARGET_PREINIT_TIMEOUT),$(CONFIG_TARGET_PREINIT_TIMEOUT),2)' >>$(1)/lib/preinit/00_preinit.conf
	echo 'pi_init_path=$(if $(CONFIG_TARGET_INIT_PATH),$(CONFIG_TARGET_INIT_PATH),"/usr/sbin:/usr/bin:/sbin:/bin")' >>$(1)/lib/preinit/00_preinit.conf
	echo 'pi_init_env=$(if $(CONFIG_TARGET_INIT_ENV),$(CONFIG_TARGET_INIT_ENV),"")' >>$(1)/lib/preinit/00_preinit.conf
	echo 'pi_init_cmd=$(if $(CONFIG_TARGET_INIT_CMD),$(CONFIG_TARGET_INIT_CMD),"/sbin/init")' >>$(1)/lib/preinit/00_preinit.conf
	echo 'pi_init_suppress_stderr="$(CONFIG_TARGET_INIT_SUPPRESS_STDERR)"' >>$(1)/lib/preinit/00_preinit.conf
	echo 'pi_ifname=$(if $(CONFIG_TARGET_PREINIT_IFNAME),$(CONFIG_TARGET_PREINIT_IFNAME),"")' >>$(1)/lib/preinit/00_preinit.conf
	echo 'pi_ip=$(if $(CONFIG_TARGET_PREINIT_IP),$(CONFIG_TARGET_PREINIT_IP),"192.168.1.1")' >>$(1)/lib/preinit/00_preinit.conf
	echo 'pi_netmask=$(if $(CONFIG_TARGET_PREINIT_NETMASK),$(CONFIG_TARGET_PREINIT_NETMASK),"255.255.255.0")' >>$(1)/lib/preinit/00_preinit.conf
	echo 'pi_broadcast=$(if $(CONFIG_TARGET_PREINIT_BROADCAST),$(CONFIG_TARGET_PREINIT_BROADCAST),"192.168.1.255")' >>$(1)/lib/preinit/00_preinit.conf
	echo 'pi_preinit_net_messages="$(CONFIG_TARGET_PREINIT_SHOW_NETMSG)"' >>$(1)/lib/preinit/00_preinit.conf
	echo 'pi_preinit_no_failsafe_netmsg="$(CONFIG_TARGET_PREINIT_SUPPRESS_FAILSAFE_NETMSG)"' >>$(1)/lib/preinit/00_preinit.conf
endef
endif

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
endef

define Build/Compile/Default

endef
Build/Compile = $(Build/Compile/Default)

ifdef CONFIG_SIGNED_PACKAGES
  define Build/Configure
	[ -s $(BUILD_KEY) -a -s $(BUILD_KEY).pub ] || \
		$(STAGING_DIR_HOST)/bin/usign -G -s $(BUILD_KEY) -p $(BUILD_KEY).pub -c "Local build key"

  endef

  define Package/base-files/install-key
	mkdir -p $(1)/etc/opkg/keys
	$(CP) $(BUILD_KEY).pub $(1)/etc/opkg/keys/`$(STAGING_DIR_HOST)/bin/usign -F -p $(BUILD_KEY).pub`

  endef
endif

define Package/base-files/install
	$(CP) ./files/* $(1)/
	$(Package/base-files/install-key)
	if [ -d $(GENERIC_PLATFORM_DIR)/base-files/. ]; then \
		$(CP) $(GENERIC_PLATFORM_DIR)/base-files/* $(1)/; \
	fi
	if [ -d $(PLATFORM_DIR)/base-files/. ]; then \
		$(CP) $(PLATFORM_DIR)/base-files/* $(1)/; \
	fi
	if [ -d $(PLATFORM_DIR)/base-files-$(PROFILE)/. ]; then \
		$(CP) $(PLATFORM_DIR)/base-files-$(PROFILE)/* $(1)/; \
	fi
	if [ -d $(PLATFORM_DIR)/$(PROFILE)/base-files/. ]; then \
		$(CP) $(PLATFORM_DIR)/$(PROFILE)/base-files/* $(1)/; \
	fi
	$(if $(filter-out $(PLATFORM_DIR),$(PLATFORM_SUBDIR)), \
		if [ -d $(PLATFORM_SUBDIR)/base-files/. ]; then \
			$(CP) $(PLATFORM_SUBDIR)/base-files/* $(1)/; \
		fi; \
		if [ -d $(PLATFORM_SUBDIR)/base-files-$(PROFILE)/. ]; then \
			$(CP) $(PLATFORM_SUBDIR)/base-files-$(PROFILE)/* $(1)/; \
		fi; \
		if [ -d $(PLATFORM_SUBDIR)/$(PROFILE)/base-files/. ]; then \
			$(CP) $(PLATFORM_SUBDIR)/$(PROFILE)/base-files/* $(1)/; \
		fi \
	)

	$(VERSION_SED) \
		$(1)/etc/banner \
		$(1)/etc/openwrt_version

	$(VERSION_SED_SCRIPT) \
		$(1)/etc/openwrt_release \
		$(1)/etc/device_info

	mkdir -p $(1)/CONTROL
	mkdir -p $(1)/dev
	mkdir -p $(1)/etc/crontabs
	mkdir -p $(1)/etc/rc.d
	mkdir -p $(1)/overlay
	mkdir -p $(1)/lib/firmware
	$(if $(LIB_SUFFIX),-$(LN) lib $(1)/lib$(LIB_SUFFIX))
	mkdir -p $(1)/mnt
	mkdir -p $(1)/proc
	mkdir -p $(1)/tmp
	mkdir -p $(1)/usr/lib
	$(if $(LIB_SUFFIX),-$(LN) lib $(1)/usr/lib$(LIB_SUFFIX))
	mkdir -p $(1)/usr/bin
	mkdir -p $(1)/sys
	mkdir -p $(1)/www
	mkdir -p $(1)/root
	$(LN) /proc/mounts $(1)/etc/mtab
	rm -f $(1)/var
	$(LN) /tmp $(1)/var
	mkdir -p $(1)/etc
	$(LN) /tmp/resolv.conf /tmp/fstab /tmp/TZ $(1)/etc/
	$(LN) /lib/functions.sh $(1)/etc/

	chmod 0600 $(1)/etc/shadow
	chmod 1777 $(1)/tmp

	$(call ImageConfigOptions,$(1))
	#$(call Package/base-files/install-target,$(1))
	for conffile in $(1)/etc/config/*; do \
		if [ -f "$$$$conffile" ]; then \
			grep "$$$${conffile##$(1)}" $(1)/CONTROL/conffiles || \
				echo "$$$${conffile##$(1)}" >> $(1)/CONTROL/conffiles; \
		fi \
	done

	## install platform dependency files
	@( \
		if [ -d ./platform-dependency-files/$(PRODUCT_NAME) ]; then \
			$(CP) ./platform-dependency-files/$(PRODUCT_NAME)/* $(1)/; \
		fi; \
		if [ -d ./platform-dependency-files/$(PRODUCT_NAME) ]; then \
			find $(1) -name .svn | $(XARGS) rm -rf; \
		fi; \
	)

endef

ifneq ($(DUMP),1)
  -include $(PLATFORM_DIR)/base-files.mk
endif

$(eval $(call BuildPackage,base-files))
