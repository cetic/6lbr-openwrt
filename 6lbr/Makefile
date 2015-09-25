include $(TOPDIR)/rules.mk

PKG_NAME:=6lbr
PKG_VERSION:=1.4
PKG_RELEASE:=0
PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/cetic/6lbr.git
PKG_SOURCE_VERSION:=develop
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION)-$(PKG_SOURCE_VERSION).tar.gz

PKG_SOURCE_SUBDIR := $(PKG_NAME)
PKG_BUILD_DIR := $(BUILD_DIR)/$(PKG_NAME)
6LBR_DIR := $(PKG_BUILD_DIR)/examples/6lbr

include $(INCLUDE_DIR)/package.mk

define Package/6lbr
  SECTION:=net
  CATEGORY:=Network
  TITLE:=CETIC 6LBR
  MAINTAINER:=Laurent <laurent.deru@cetic.be>
  DEPENDS:=+libncurses +kmod-tun +kmod-usb-serial-ftdi +logrotate
endef

define Package/6lbr/description
6LBR
endef

define Build/Configure
endef

define Build/Compile
	mkdir -p $(6LBR_DIR)/bin
	CFLAGS="$(TARGET_CFLAGS)" LDFLAGS="$(TARGET_LDFLAGS)" $(MAKE) -C $(6LBR_DIR) \
		TARGET=native CC="$(TARGET_CC)" LD="$(TARGET_CC)" AR="$(TARGET_AR)" V=1 CETIC_6LBR_ROUTER=1 WITH_UDPCLIENT=0
	mv $(6LBR_DIR)/cetic-6lbr.native $(6LBR_DIR)/bin/cetic_6lbr_router
	CFLAGS="$(TARGET_CFLAGS)" LDFLAGS="$(TARGET_LDFLAGS)" $(MAKE) -C $(6LBR_DIR) \
		TARGET=native CC="$(TARGET_CC)" LD="$(TARGET_CC)" AR="$(TARGET_AR)" V=1 CETIC_6LBR_SMARTBRIDGE=1 WITH_UDPCLIENT=0
	mv $(6LBR_DIR)/cetic-6lbr.native $(6LBR_DIR)/bin/cetic_6lbr_smart_bridge
	CFLAGS="$(TARGET_CFLAGS)" LDFLAGS="$(TARGET_LDFLAGS)" $(MAKE) -C $(6LBR_DIR) \
		CC="$(TARGET_CC)" LD="$(TARGET_CC)" AR="$(TARGET_AR)" V=1 CETIC_6LBR_6LR=1 WITH_UDPCLIENT=1 WITH_UDPSERVER=0
	mv $(6LBR_DIR)/cetic-6lbr.native $(6LBR_DIR)/bin/cetic_6lbr_6lr
	CFLAGS="$(TARGET_CFLAGS)" LDFLAGS="$(TARGET_LDFLAGS)" $(MAKE) -C $(6LBR_DIR) \
                TARGET=native CC="$(TARGET_CC)" LD="$(TARGET_CC)" AR="$(TARGET_AR)" V=1 plugins
	CFLAGS="$(TARGET_CFLAGS)" LDFLAGS="$(TARGET_LDFLAGS)" $(MAKE) -C $(6LBR_DIR)/tools \
                CC="$(TARGET_CC)" LD="$(TARGET_CC)" AR="$(TARGET_AR)" V=1 
endef

define Package/6lbr/install
	$(INSTALL_DIR) $(1)/etc/6lbr
	cp -r $(6LBR_DIR)/package/etc/6lbr $(1)/etc
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) $(6LBR_DIR)/openwrt/init.d/* $(1)/etc/init.d/
	$(INSTALL_DIR) $(1)/etc/logrotate.d
	$(INSTALL_CONF) $(6LBR_DIR)/openwrt/logrotate.d/* $(1)/etc/logrotate.d/
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(6LBR_DIR)/package/usr/bin/* $(1)/usr/bin/
	$(INSTALL_DIR) $(1)/usr/lib/6lbr
	cp -r $(6LBR_DIR)/package/usr/lib/6lbr $(1)/usr/lib
	$(INSTALL_DIR) $(1)/usr/lib/6lbr/bin
	$(INSTALL_BIN) $(6LBR_DIR)/bin/* $(1)/usr/lib/6lbr/bin
	$(INSTALL_BIN) $(6LBR_DIR)/tools/nvm_tool $(1)/usr/lib/6lbr/bin
	$(MAKE) -C $(6LBR_DIR) DESTDIR=$(1) plugins-install
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
