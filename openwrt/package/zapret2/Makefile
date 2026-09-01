#
# Copyright (C) 2024-2026 bol-van (upstream) / carp4 (packaging)
# Complete OpenWrt package for bol-van/zapret2 (nfqws2).
#
# Builds the nfqws2 engine from source and ships the full runtime
# (Lua strategy libs, init scripts, default config, ipset lists, fake blobs)
# under /opt/zapret2 so that /etc/init.d/zapret2 is self-contained.
#
# Placement: this Makefile lives at <repo>/openwrt/package/zapret2/Makefile.
# When the whole repo is copied into an OpenWrt SDK as package/zapret2/,
# the repo root (where nfq2/, lua/, common/, ... live) is $(CURDIR)/../..
#
include $(TOPDIR)/rules.mk

PKG_NAME:=zapret2
PKG_RELEASE:=1
PKG_VERSION:=72

REPO_ROOT:=$(CURDIR)/../..

include $(INCLUDE_DIR)/package.mk

LUA_JIT?=1
ifeq ($(LUA_JIT),1)
LUAJIT_VER?=2.1
LUA_VER?=5.1
LUA_DEP:=luajit
LUA_INCLUDE:=-I$(STAGING_DIR)/usr/include/luajit-$(LUAJIT_VER)
LUA_LIBRARY:=-L$(STAGING_DIR)/usr/lib -lluajit-$(LUA_VER)
else
LUA_VER?=5.3
LUA_DEP:=lua$(LUA_VER)
LUA_INCLUDE:=-I$(STAGING_DIR)/usr/include/lua$(LUA_VER)
LUA_LIBRARY:=-L$(STAGING_DIR)/usr/lib -llua$(LUA_VER)
endif

# runtime base on target
ZAPRET_DIR:=/opt/zapret2

define Package/zapret2
	SECTION:=net
	CATEGORY:=Network
	TITLE:=zapret2 - anti-DPI engine (nfqws2) with full runtime
	SUBMENU:=Zapret
	URL:=https://github.com/bol-van/zapret2
	DEPENDS:=+libnetfilter-queue1 +libmnl0 +libcap +zlib +$(LUA_DEP)
endef

define Package/zapret2/description
	zapret2 (nfqws2) is the second-generation anti-DPI engine: a
	programmable packet manipulation tool with Lua-based DPI-bypass
	strategies. This package builds the engine and installs the complete
	runtime (Lua strategy library, init scripts, default config, ipset
	lists and fake blobs) to $(ZAPRET_DIR).
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) $(REPO_ROOT)/nfq2/* $(PKG_BUILD_DIR)/
endef

define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR) $(TARGET_CONFIGURE_OPTS) \
		LUA_CFLAGS="$(LUA_INCLUDE)" LUA_LIB="$(LUA_LIBRARY)"
endef

define Package/zapret2/install
	# engine binary
	$(INSTALL_DIR) $(1)$(ZAPRET_DIR)/nfq2
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/nfqws2 $(1)$(ZAPRET_DIR)/nfq2/

	# lua strategy library
	$(INSTALL_DIR) $(1)$(ZAPRET_DIR)/lua
	$(INSTALL_DATA) $(REPO_ROOT)/lua/*.lua $(1)$(ZAPRET_DIR)/lua/

	# common runtime helpers
	$(INSTALL_DIR) $(1)$(ZAPRET_DIR)/common
	$(INSTALL_BIN) $(REPO_ROOT)/common/*.sh $(1)$(ZAPRET_DIR)/common/

	# ip2net helper + ipset scripts/lists
	$(INSTALL_DIR) $(1)$(ZAPRET_DIR)/ipset
	$(CP) $(REPO_ROOT)/ipset/* $(1)$(ZAPRET_DIR)/ipset/

	# fake blobs
	$(INSTALL_DIR) $(1)$(ZAPRET_DIR)/files
	$(CP) $(REPO_ROOT)/files/* $(1)$(ZAPRET_DIR)/files/

	# blockcheck strategy tree (reference / on-device blockcheck)
	$(INSTALL_DIR) $(1)$(ZAPRET_DIR)/blockcheck2.d
	$(CP) $(REPO_ROOT)/blockcheck2.d/* $(1)$(ZAPRET_DIR)/blockcheck2.d/
	$(INSTALL_BIN) $(REPO_ROOT)/blockcheck2.sh $(1)$(ZAPRET_DIR)/blockcheck2.sh

	# default config
	$(INSTALL_DATA) $(REPO_ROOT)/config.default $(1)$(ZAPRET_DIR)/config.default

	# init scripts (procd) + firewall
	$(INSTALL_DIR) $(1)$(ZAPRET_DIR)/init.d/openwrt
	$(CP) $(REPO_ROOT)/init.d/openwrt/* $(1)$(ZAPRET_DIR)/init.d/openwrt/
	$(INSTALL_BIN) $(REPO_ROOT)/init.d/openwrt/zapret2 $(1)/etc/init.d/zapret2
	$(INSTALL_BIN) $(REPO_ROOT)/init.d/openwrt/90-zapret2 $(1)/etc/hotplug.d/iface/90-zapret2

	# create runtime config from default on first boot
	$(INSTALL_DIR) $(1)/etc/uci-defaults
	$(INSTALL_BIN) $(REPO_ROOT)/openwrt/uci-defaults.zapret2 $(1)/etc/uci-defaults/99-zapret2-config
endef

$(eval $(call BuildPackage,zapret2))
