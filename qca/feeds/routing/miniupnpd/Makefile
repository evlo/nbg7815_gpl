#
# Copyright (C) 2006-2014 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=miniupnpd
PKG_VERSION:=2.0
PKG_RELEASE:=3

PKG_SOURCE_URL:=http://miniupnp.free.fr/files
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_MD5SUM:=1c07a215dd9b362e75a9efc05e2fb3b4

PKG_MAINTAINER:=Markus Stenberg <fingon@iki.fi>
PKG_LICENSE:=BSD-3-Clause

PATCH_DIR:=patches-$(CONFIG_ZYXEL_PROJECT_NAME)

include $(INCLUDE_DIR)/package.mk

define Package/miniupnpd
  SECTION:=net
  CATEGORY:=Network
  DEPENDS:=+iptables +libip4tc +IPV6:libip6tc +IPV6:ip6tables +libuuid
  TITLE:=Lightweight UPnP IGD, NAT-PMP & PCP daemon
  SUBMENU:=Firewall
  URL:=http://miniupnp.free.fr/
endef

define Package/miniupnpd/conffiles
/etc/config/upnpd
endef

define Package/miniupnpd/postinst
#!/bin/sh

if [ -z "$$IPKG_INSTROOT" ]; then
  ( . /etc/uci-defaults/99-miniupnpd )
  rm -f /etc/uci-defaults/99-miniupnpd
fi

exit 0
endef

define Build/Prepare
	$(call Build/Prepare/Default)
	echo "OpenWrt" | tr \(\)\  _ >$(PKG_BUILD_DIR)/os.openwrt
endef

MAKE_FLAGS += \
	TARGET_OPENWRT=1 TEST=0 \
	LIBS="" \
	CC="$(TARGET_CC) -DIPTABLES_143 \
		-lip4tc $(if $(CONFIG_IPV6),-lip6tc) -luuid" \
	CONFIG_OPTIONS="$(if $(CONFIG_IPV6),--ipv6) --leasefile" \
	-f Makefile.linux \
	miniupnpd

define Package/miniupnpd/install
	$(INSTALL_DIR) $(1)/usr/sbin $(1)/etc/init.d $(1)/etc/config $(1)/etc/uci-defaults $(1)/etc/hotplug.d/iface $(1)/usr/share/miniupnpd
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/miniupnpd $(1)/usr/sbin/miniupnpd

	$(INSTALL_BIN) ./files/miniupnpd.init $(1)/etc/init.d/miniupnpd
	$(INSTALL_CONF) ./files/upnpd.config $(1)/etc/config/upnpd
	$(INSTALL_DATA) ./files/miniupnpd.hotplug $(1)/etc/hotplug.d/iface/14-miniupnpd
	$(INSTALL_DIR) $(1)/sbin
	$(INSTALL_BIN) ./files/act_miniupnpd $(1)/sbin/

endef

$(eval $(call BuildPackage,miniupnpd))
