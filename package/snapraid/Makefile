#
# Copyright (C) 2026 aseracorp
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=snapraid
PKG_VERSION:=14.9
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://github.com/amadvance/snapraid/releases/download/v$(PKG_VERSION)
PKG_HASH:=40c216979d9d9853248060497341f74feaa07c8ae15927b6b14972c4f9d143d5

PKG_MAINTAINER:=aseracorp <noreply@aseracorp.dev>
PKG_LICENSE:=GPL-3.0-or-later
PKG_LICENSE_FILES:=COPYING
PKG_CPE_ID:=cpe:/a:amadvance:snapraid

PKG_INSTALL:=1

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/autotools.mk

# SnapRAID auto-detects smartctl at runtime, so no hard dependency.
# Build lean: disable optional libblkid.
CONFIGURE_ARGS += \
	--without-blkid

# SnapRAID uses 64-bit atomic operations; on 32-bit MIPS/ARM (and other archs
# without native 64-bit atomics) these need libatomic. Linking it always is
# harmless where native atomics exist. LIBS is placed after the object files
# by the generated Makefile, which is what a static link of libatomic needs.
CONFIGURE_VARS += \
	LIBS="-latomic"

define Package/snapraid
  SECTION:=utils
  CATEGORY:=Utilities
  SUBMENU:=Filesystem
  TITLE:=Disk array backup with parity protection
  URL:=https://www.snapraid.it/
  # libatomic.so.1 is required at runtime on archs without native 64-bit
  # atomics (e.g. 32-bit MIPS). Package the dependency so apk resolves it.
  DEPENDS:=+libatomic
endef

define Package/snapraid/description
	SnapRAID is a backup program for disk arrays. It stores parity
	information of your data and it recovers from up to six disk
	failures.
	SnapRAID is mainly targeted for a home media center, with a lot
	of big files that rarely change.
endef

# Install the binary and upstream English man pages (snapraid.1,
# snapraid_log.1). OpenWrt SDK installs man pages under usr/man/.
define Package/snapraid/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/snapraid $(1)/usr/bin/
	$(INSTALL_DIR) $(1)/usr/man/man1
	$(INSTALL_DATA) $(PKG_INSTALL_DIR)/usr/man/man1/snapraid.1 $(1)/usr/man/man1/
	$(INSTALL_DATA) $(PKG_INSTALL_DIR)/usr/man/man1/snapraid_log.1 $(1)/usr/man/man1/
endef

$(eval $(call BuildPackage,snapraid))
