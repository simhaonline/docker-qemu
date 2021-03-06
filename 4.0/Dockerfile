FROM debian:buster-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
		ovmf \
	&& rm -rf /var/lib/apt/lists/*

COPY *.patch /qemu-patches/

# https://wiki.qemu.org/SecurityProcess
ENV QEMU_KEYS \
# Michael Roth
		CEACC9E15534EBABB82D3FA03353C9CEF108B584
# https://wiki.qemu.org/Planning/ReleaseProcess#Sign_the_resulting_tarball_with_GPG: (they get signed by whoever is making the release)

# https://www.qemu.org/download/#source
# https://download.qemu.org/?C=M;O=D
ENV QEMU_VERSION 4.0.1
ENV QEMU_URL https://download.qemu.org/qemu-4.0.1.tar.xz

RUN set -eux; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		ca-certificates \
		gnupg dirmngr \
		wget \
		xz-utils \
		\
		patch \
		\
		bzip2 \
		gcc \
		gnutls-dev \
		libaio-dev \
		libbz2-dev \
		libc-dev \
		libcap-dev \
		libcap-ng-dev \
		libcurl4-gnutls-dev \
		libglib2.0-dev \
		libiscsi-dev \
		libjpeg-dev \
		libncursesw5-dev \
		libnfs-dev \
		libnuma-dev \
		libpixman-1-dev \
		libpng-dev \
		libseccomp-dev \
		libssh2-1-dev \
		libusb-1.0-0-dev \
		libusbredirparser-dev \
		libxen-dev \
		make \
		pkg-config \
		python \
		xfslibs-dev \
		zlib1g-dev \
	; \
	rm -rf /var/lib/apt/lists/*; \
	\
	tarball="$(basename "$QEMU_URL")"; \
	wget -O "$tarball.sig" "$QEMU_URL.sig"; \
	wget -O "$tarball" "$QEMU_URL" --progress=dot:giga; \
	\
	export GNUPGHOME="$(mktemp -d)"; \
	for key in $QEMU_KEYS; do \
		gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
	done; \
	gpg --batch --verify "$tarball.sig" "$tarball"; \
	gpgconf --kill all; \
	rm -rf "$GNUPGHOME"; \
	\
	mkdir /usr/src/qemu; \
	tar -xf "$tarball" -C /usr/src/qemu --strip-components=1; \
	rm "$tarball" "$tarball.sig"; \
	\
	cd /usr/src/qemu; \
	\
	for p in /qemu-patches/*.patch; do \
		patch --strip 1 --input "$p"; \
	done; \
	rm -rf /qemu-patches; \
	\
	./configure --help; \
	./configure \
		--target-list=' \
# system targets
# (https://sources.debian.org/src/qemu/stretch/debian/rules/#L57-L61, slimmed)
			i386-softmmu x86_64-softmmu aarch64-softmmu arm-softmmu m68k-softmmu \
			mips64-softmmu mips64el-softmmu ppc64-softmmu \
			sparc64-softmmu s390x-softmmu \
# user targets
# (https://sources.debian.org/src/qemu/stretch/debian/rules/#L81-L86, slimmed)
			i386-linux-user x86_64-linux-user aarch64-linux-user arm-linux-user m68k-linux-user \
			mips64-linux-user mips64el-linux-user \
			ppc64-linux-user ppc64le-linux-user sparc64-linux-user \
			s390x-linux-user \
		' \
		--disable-docs \
		--disable-gtk --disable-vte \
		--disable-sdl \
		--enable-attr \
		--enable-bzip2 \
		--enable-cap-ng \
		--enable-curl \
		--enable-curses \
		--enable-fdt \
		--enable-gnutls \
		--enable-kvm \
		--enable-libiscsi \
		--enable-libnfs \
		--enable-libssh2 \
		--enable-libusb \
		--enable-linux-aio \
		--enable-linux-user \
		--enable-modules \
		--enable-numa \
		--enable-seccomp \
		--enable-system \
		--enable-tools \
		--enable-usb-redir \
		--enable-vhost-net \
		--enable-vhost-user \
		--enable-vhost-vsock \
		--enable-virtfs \
		--enable-vnc \
		--enable-vnc-jpeg \
		--enable-vnc-png \
		--enable-xen \
		--enable-xfsctl \
#		--enable-rbd \
#		--enable-vde \
	; \
	make -j "$(nproc)"; \
	make install; \
	\
	cd /; \
	rm -rf /usr/src/qemu; \
	\
	apt-mark auto '.*' > /dev/null; \
	[ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; \
	find /usr/local -type f -executable -exec ldd '{}' ';' \
		| awk '/=>/ { print $(NF-1) }' \
		| sort -u \
		| xargs -r dpkg-query --search \
		| cut -d: -f1 \
		| sort -u \
		| xargs -r apt-mark manual \
	; \
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	\
# basic smoke test
	qemu-img --version

STOPSIGNAL SIGHUP

EXPOSE 22
EXPOSE 5900

COPY start-qemu /usr/local/bin/
CMD ["start-qemu"]
