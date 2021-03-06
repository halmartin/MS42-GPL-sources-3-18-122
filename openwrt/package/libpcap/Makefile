# $Id: Makefile 3139 2006-02-05 17:30:49Z florian $

include $(TOPDIR)/rules.mk

PKG_NAME:=libpcap
PKG_VERSION:=1.8.1
PKG_RELEASE:=1
PKG_MD5SUM:=3d48f9cd171ff12b0efd9134b52f1447

PKG_SOURCE_URL:=https://dl.meraki.net/
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_CAT:=zcat

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)
PKG_INSTALL_DIR:=$(PKG_BUILD_DIR)/ipkg-install

include $(TOPDIR)/package/rules.mk

$(eval $(call PKG_template,LIBPCAP,libpcap,$(PKG_VERSION)-$(PKG_RELEASE),$(ARCH)))

$(PKG_BUILD_DIR)/.configured:
	(cd $(PKG_BUILD_DIR); rm -rf config.{cache,status} ; \
		$(TARGET_CONFIGURE_OPTS) \
		CFLAGS="$(TARGET_CFLAGS)" \
		CPPFLAGS="-I$(STAGING_DIR)/usr/include" \
		LDFLAGS="-L$(STAGING_DIR)/usr/lib" \
		ac_cv_linux_vers=$(BR2_DEFAULT_KERNEL_HEADERS) \
		./configure \
		  --target=$(GNU_TARGET_NAME) \
		  --host=$(GNU_TARGET_NAME) \
		  --build=$(GNU_HOST_NAME) \
		  --program-prefix="" \
		  --program-suffix="" \
		  --prefix=/usr \
		  --exec-prefix=/usr \
		  --bindir=/usr/bin \
		  --datadir=/usr/share \
		  --includedir=/usr/include \
		  --infodir=/usr/share/info \
		  --libdir=/usr/lib \
		  --libexecdir=/usr/lib \
		  --localstatedir=/var \
		  --mandir=/usr/share/man \
		  --sbindir=/usr/sbin \
		  --sysconfdir=/etc \
		  $(DISABLE_NLS) \
		  $(DISABLE_LARGEFILE) \
		  --enable-shared \
		  --enable-static \
		  --disable-yydebug \
		  --enable-ipv6 \
		  --with-build-cc=$(HOSTCC) \
		  --with-pcap=linux \
	);
	touch $@

$(PKG_BUILD_DIR)/.built:
	rm -rf $(PKG_INSTALL_DIR)
	mkdir -p $(PKG_INSTALL_DIR)
	$(MAKE) -C $(PKG_BUILD_DIR) \
		CCOPT="$(TARGET_CFLAGS) -fPIC -I$(STAGING_DIR)/include -I$(STAGING_DIR)/usr/include" \
		CC="$(TARGET_CC)" DESTDIR="$(PKG_INSTALL_DIR)" \
		all shared
	$(MAKE) -C $(PKG_BUILD_DIR) \
		CCOPT="$(TARGET_CFLAGS) -fPIC -I$(STAGING_DIR)/include -I$(STAGING_DIR)/usr/include" \
		CC="$(TARGET_CC)" DESTDIR="$(PKG_INSTALL_DIR)" \
		install install-shared
	touch $@

$(IPKG_LIBPCAP):
	install -m0755 -d $(IDIR_LIBPCAP)/usr/lib
	$(CP) -a $(PKG_INSTALL_DIR)/usr/lib/libpcap.so* $(IDIR_LIBPCAP)/usr/lib/
	$(RSTRIP) $(IDIR_LIBPCAP)
	$(IPKG_BUILD) $(IDIR_LIBPCAP) $(PACKAGE_DIR)

$(STAGING_DIR)/usr/lib/libpcap.so: $(PKG_BUILD_DIR)/.built
	rm -f $@
	mkdir -p $(STAGING_DIR)/usr/include
	$(CP) $(PKG_INSTALL_DIR)/usr/bin/pcap* $(STAGING_DIR)/usr/bin/
	$(CP) $(PKG_INSTALL_DIR)/usr/include/pcap* $(STAGING_DIR)/usr/include/
	mkdir -p $(STAGING_DIR)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libpcap.* $(STAGING_DIR)/usr/lib/

install-dev: $(STAGING_DIR)/usr/lib/libpcap.so

uninstall-dev:
	rm -rf $(STAGING_DIR)/usr/include/pcap*
	rm -rf $(STAGING_DIR)/usr/lib/libpcap.{a,so*}

compile-targets: install-dev
clean-targets: uninstall-dev
