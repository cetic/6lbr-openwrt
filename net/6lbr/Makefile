include $(TOPDIR)/rules.mk

PKG_NAME:=6lbr
PKG_VERSION:=1.4.2
PKG_RELEASE:=0
PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/cetic/6lbr.git
PKG_SOURCE_VERSION:=6lbr-1.4.2
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION)-$(PKG_SOURCE_VERSION).tar.gz
PKG_LICENSE:=BSD-3c
PKG_LICENSE_FILES:=LICENSE
PKG_SOURCE_SUBDIR:=$(PKG_NAME)
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)


include $(INCLUDE_DIR)/package.mk

define Package/6lbr
  SECTION:=net
  CATEGORY:=Network
  TITLE:=CETIC 6LoWPAN Border Router
  MAINTAINER:=Laurent <laurent.deru@cetic.be>
  DEPENDS:=+libncurses +kmod-tun +kmod-usb-serial +kmod-usb-serial-ftdi +kmod-usb-acm +logrotate +librt
endef

define Package/6lbr/description
CETIC 6LBR is an enhanced version of Contiki's 6LoWPAN Border Router solution.
It provides network configuration and monitoring, and supports out of the box redundant and distributed operating modes.
endef

define Package/6lbr/config
	source "$(SOURCE)/Config.in"
endef

define Build/Compile
	mkdir -p $(PKG_BUILD_DIR)/examples/6lbr/bin
	$(MAKE) -C $(PKG_BUILD_DIR)/examples/6lbr clean
	CFLAGS="$(TARGET_CFLAGS)" LDFLAGS="$(TARGET_LDFLAGS)" $(MAKE) -C $(PKG_BUILD_DIR)/examples/6lbr \
                TARGET=native CC="$(TARGET_CC)" LD="$(TARGET_CC)" AR="$(TARGET_AR)" V=1 CETIC_6LBR_ROUTER=1 WITH_UDPCLIENT=0
	mv $(PKG_BUILD_DIR)/examples/6lbr/cetic-6lbr.native $(PKG_BUILD_DIR)/examples/6lbr/bin/cetic_6lbr_router
	$(MAKE) -C $(PKG_BUILD_DIR)/examples/6lbr clean
	CFLAGS="$(TARGET_CFLAGS)" LDFLAGS="$(TARGET_LDFLAGS)" $(MAKE) -C $(PKG_BUILD_DIR)/examples/6lbr \
                TARGET=native CC="$(TARGET_CC)" LD="$(TARGET_CC)" AR="$(TARGET_AR)" V=1 CETIC_6LBR_SMARTBRIDGE=1 WITH_UDPCLIENT=0
	mv $(PKG_BUILD_DIR)/examples/6lbr/cetic-6lbr.native $(PKG_BUILD_DIR)/examples/6lbr/bin/cetic_6lbr_smart_bridge
	$(MAKE) -C $(PKG_BUILD_DIR)/examples/6lbr clean
	CFLAGS="$(TARGET_CFLAGS)" LDFLAGS="$(TARGET_LDFLAGS)" $(MAKE) -C $(PKG_BUILD_DIR)/examples/6lbr \
                CC="$(TARGET_CC)" LD="$(TARGET_CC)" AR="$(TARGET_AR)" V=1 CETIC_6LBR_6LR=1 WITH_UDPCLIENT=1 WITH_UDPSERVER=0
	mv $(PKG_BUILD_DIR)/examples/6lbr/cetic-6lbr.native $(PKG_BUILD_DIR)/examples/6lbr/bin/cetic_6lbr_6lr
ifeq ($(CONFIG_SIXLBR_COMPILE_PLUGINS),y)
	$(MAKE) -C $(PKG_BUILD_DIR)/examples/6lbr plugins-clean
	CFLAGS="$(TARGET_CFLAGS)" LDFLAGS="$(TARGET_LDFLAGS)" $(MAKE) -C $(PKG_BUILD_DIR)/examples/6lbr \
                TARGET=native CC="$(TARGET_CC)" LD="$(TARGET_CC)" AR="$(TARGET_AR)" V=1 plugins
endif
	$(MAKE) -C $(PKG_BUILD_DIR)/examples/6lbr/tools clean
	CFLAGS="$(TARGET_CFLAGS)" LDFLAGS="$(TARGET_LDFLAGS)" $(MAKE) -C $(PKG_BUILD_DIR)/examples/6lbr/tools \
                CC="$(TARGET_CC)" LD="$(TARGET_CC)" AR="$(TARGET_AR)" V=1
endef

define Package/6lbr/install
	$(INSTALL_DIR) $(1)/etc/6lbr
	cp -r $(PKG_BUILD_DIR)/examples/6lbr/package/etc/6lbr $(1)/etc
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/examples/6lbr/openwrt/init.d/* $(1)/etc/init.d/
	$(INSTALL_DIR) $(1)/etc/logrotate.d
	$(INSTALL_CONF) $(PKG_BUILD_DIR)/examples/6lbr/openwrt/logrotate.d/* $(1)/etc/logrotate.d/
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/examples/6lbr/package/usr/bin/* $(1)/usr/bin/
	$(INSTALL_DIR) $(1)/usr/lib/6lbr
	cp -r $(PKG_BUILD_DIR)/examples/6lbr/package/usr/lib/6lbr $(1)/usr/lib
	$(INSTALL_DIR) $(1)/usr/lib/6lbr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/examples/6lbr/bin/* $(1)/usr/lib/6lbr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/examples/6lbr/tools/nvm_tool $(1)/usr/lib/6lbr/bin
ifeq ($(CONFIG_SIXLBR_COMPILE_PLUGINS),y)
	$(MAKE) -C $(PKG_BUILD_DIR)/examples/6lbr DESTDIR=$(1) plugins-install
endif
endef

define Package/6lbr/postinst
#!/bin/sh
# check if we are on real system
if [ -z "$${IPKG_INSTROOT}" ]; then
        echo "Enabling rc.d symlink for 6lbr"
        /etc/init.d/6lbr enable
        (crontab -l -u root 2>/dev/null; echo "# 01 * * * * logrotate /etc/logrotate.d/6lbr.conf") | crontab -u root -
        echo "Please, edit file /etc/6lbr/6lbr.conf and enable cron job!"
fi
exit 0
endef

define Package/6lbr/prerm
#!/bin/sh
# check if we are on real system
if [ -z "$${IPKG_INSTROOT}" ]; then
        echo "Removing rc.d symlink for 6lbr"
        /etc/init.d/6lbr disable
        echo "Don't forget to disable cron job!"
fi
exit 0
endef

$(eval $(call BuildPackage,6lbr))
