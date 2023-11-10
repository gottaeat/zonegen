include $(TOPDIR)/rules.mk

PKG_NAME:=zonegen
PKG_RELEASE:=2

PKG_SOURCE_URL:=https://github.com/gottaeat/zonegen.git
PKG_SOURCE_PROTO:=git
PKG_SOURCE_VERSION:=7d56d24ec5ba5a12a9754ec9ab2c54359a13fb40

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/meson.mk

define Package/zonegen
	SECTION:=net
	CATEGORY:=Network
	TITLE:=zonegen
	DEPENDS:=+bind-dig +bind-rndc +bind-server +odhcpd
	URL:=https://github.com:gottaeat/zonegen
endef

define Package/zonegen/install
	$(INSTALL_DIR) $(1)/bin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/bin/* $(1)/bin/
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/etc/init.d/* $(1)/etc/init.d/
endef

$(eval $(call BuildPackage,zonegen))
