# custom postgresql container image, fit for synth.download's requirement
# based on the pgroonga image, with pg_repack inject

FROM groonga/pgroonga:latest-alpine-18

ENV DOCKER_PG_LLVM_DEPS="llvm21-dev clang21"
ENV PG_REPACK_VERSION=1.5.3

RUN set -eux; \
	apk add --no-cache --virtual .build-deps \
		$DOCKER_PG_LLVM_DEPS \
		bison \
		coreutils \
		dpkg-dev dpkg \
		flex \
		g++ \
		gcc \
		krb5-dev \
		libc-dev \
		libedit-dev \
		libxml2-dev \
		libxslt-dev \
		linux-headers \
		make \
		openldap-dev \
		openssl-dev \
		perl-dev \
		perl-ipc-run \
		perl-utils \
		python3-dev \
		tcl-dev \
		util-linux-dev \
		zlib-dev \
		icu-dev \
		lz4-dev \
		zstd-dev \
		readline-dev \
		liburing-dev \
		curl-dev \
	; \
	cd /usr/src; \
	wget -O pg_repack.zip http://api.pgxn.org/dist/pg_repack/${PG_REPACK_VERSION}/pg_repack-${PG_REPACK_VERSION}.zip \
	&& unzip pg_repack.zip || true && rm -f pg_repack.zip \
	&& cd pg_repack-${PG_REPACK_VERSION} \
	&& make \
	&& make install; \
	apk del --no-network .build-deps; \
	cd /; \
	rm -rf \
		/usr/src/pg_repack-${PG_REPACK_VERSION} \
		/usr/local/share/doc \
		/usr/local/share/man \
	; \
	\
	pg_repack --version