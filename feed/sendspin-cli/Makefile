#
# OpenWrt / OpenLumi package: Sendspin headless player (sendspin-cpp-cli)
# Target: Xiaomi DGNWG05LM, OpenLumi 25.12.x (arch arm_cortex-a7_neon-vfpv4, musl, GCC 14)
#

include $(TOPDIR)/rules.mk

PKG_NAME:=sendspin-cli
PKG_VERSION:=0.3.0
PKG_RELEASE:=2

PKG_SOURCE:=v$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://github.com/Sendspin/sendspin-cpp-cli/archive/refs/tags/
PKG_HASH:=baa9febbc08d58e1780d6a62fb0b20d458cd1a3cbb177929ba5f4ff0b01d335c
PKG_BUILD_DIR:=$(BUILD_DIR)/sendspin-cpp-cli-$(PKG_VERSION)

PKG_LICENSE:=Apache-2.0
PKG_LICENSE_FILES:=LICENSE

PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/cmake.mk

define Package/sendspin-cli
	SECTION:=sound
	CATEGORY:=Sound
	TITLE:=Sendspin synchronized audio player (headless, ALSA)
	URL:=https://github.com/Sendspin/sendspin-cpp-cli
	DEPENDS:=+libstdcpp +libatomic +alsa-lib +zlib +mdnsd +libavahi-client +libopenssl +libsodium
endef

define Package/sendspin-cli/description
	Headless Sendspin (Open Home Foundation) player built on sendspin-cpp.
	Plays synchronized multi-room audio from Music Assistant through ALSA.
endef

define Package/sendspin-cli/conffiles
/etc/sendspin-cli.conf
endef

# Only the ALSA backend is wanted on the gateway. mDNS uses OpenWrt's mdnsd package,
# which provides dns_sd.h/libdns_sd through the SDK staging directory and the runtime
# mDNS responder on the gateway.
# NOTE: sendspin-cpp and its dependencies are fetched by CMake FetchContent during
# configure, so the build machine needs network access (fine on GitHub Actions).
CMAKE_OPTIONS += \
	-DSENDSPIN_CLI_BUILD_TESTS=OFF \
	-DSENDSPIN_CLI_WITH_ALSA=ON \
	-DSENDSPIN_CLI_WITH_PORTAUDIO=OFF \
	-DSENDSPIN_CLI_WITH_PULSE=OFF \
	-DSENDSPIN_CLI_WITH_PIPEWIRE=OFF \
	-DSENDSPIN_CLI_WITH_MDNS=ON \
	-DCMAKE_BUILD_TYPE=Release

TARGET_LDFLAGS += -latomic

define Package/sendspin-cli/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(CMAKE_BINARY_DIR)/sendspin-cli $(1)/usr/bin/sendspin-cli
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/sendspin-cli.init $(1)/etc/init.d/sendspin-cli
	$(INSTALL_CONF) ./files/sendspin-cli.conf $(1)/etc/sendspin-cli.conf
endef

$(eval $(call BuildPackage,sendspin-cli))
