FROM xutl/ubuntu:xenial

LABEL maintainer="xutongle@gmail.com"

ARG HTTPD_VERSION
# Environment settings
ENV HTTPD_VERSION=${HTTPD_VERSION:-2.2.34}
ENV HTTPD_TGZ_URL http://mirrors.hust.edu.cn/apache/httpd/httpd-${HTTPD_VERSION}.tar.gz
ENV HTTPD_PREFIX /usr/local/apache2
ENV PATH $HTTPD_PREFIX/bin:$PATH

RUN set -xe \
	&& buildDeps=" \
		libpcre3-dev \
		libzip-dev \
		libcurl4-openssl-dev \
		libssl-dev \
		libexpat1-dev \
	" \
#	&& sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/' /etc/apt/sources.list \
	&& apt-get update && apt-get install -y --no-install-recommends --no-install-suggests libpcre3 libzip4 expat $buildDeps && rm -r /var/lib/apt/lists/* \
	\
	&& AprVersion=1.6.3 \
		AprUtilVersion=1.6.1 \
		nghttp2Version=1.29.0 \
	&& cd /usr/local/src \
	&& curl -fSL https://github.com/nghttp2/nghttp2/releases/download/v${nghttp2Version}/nghttp2-${nghttp2Version}.tar.gz -o nghttp2-${nghttp2Version}.tar.gz \
	&& tar zxf nghttp2-${nghttp2Version}.tar.gz \
	&& rm -rf nghttp2-${nghttp2Version}.tar.gz \
	&& cd nghttp2-${nghttp2Version} \
	&& ./configure --prefix=/usr/local/nghttp2 \
	&& make -j "$(nproc)" \
	&& make install \
	&& cd /usr/local/src \
	&& rm -rf /usr/local/src/nghttp2-${nghttp2Version} \
	&& curl -fSL ${HTTPD_TGZ_URL} -o httpd-${HTTPD_VERSION}.tar.gz \
	&& tar zxf httpd-${HTTPD_VERSION}.tar.gz \
	&& rm -rf httpd-${HTTPD_VERSION}.tar.gz \
	&& cd httpd-${HTTPD_VERSION}/srclib \
	&& curl -fSL http://mirrors.hust.edu.cn/apache/apr/apr-${AprVersion}.tar.gz -o apr-${AprVersion}.tar.gz \
	&& curl -fSL http://mirrors.hust.edu.cn/apache/apr/apr-util-${AprUtilVersion}.tar.gz -o apr-util-${AprUtilVersion}.tar.gz \
	&& tar zxf apr-${AprVersion}.tar.gz \
	&& tar zxf apr-util-${AprUtilVersion}.tar.gz \
	&& mv apr-${AprVersion} apr \
	&& mv apr-util-${AprUtilVersion} apr-util \
	&& rm -f apr-${AprVersion}.tar.gz \
	&& rm -f apr-util-${AprUtilVersion}.tar.gz \
	&& cd /usr/local/src/httpd-${HTTPD_VERSION}/ \
	&& gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
	&& ./configure \
		--build="$gnuArch" \
		--prefix="$HTTPD_PREFIX" \
		--enable-mods-shared=most \
		--enable-headers \
		--enable-mime-magic \
		--enable-proxy \
		--enable-so \
		--enable-rewrite \
		--enable-ssl \
		--enable-deflate \
		--with-pcre \
		--with-included-apr \
		--with-apr-util \
		--enable-mpms-shared=all \
		--enable-remoteip \
		--enable-http2 \
		--with-nghttp2=/usr/local/nghttp2
	&& make -j "$(nproc)" \
	&& make install \
	&& rm -rf /usr/local/src/httpd-${HTTPD_VERSION} \
	&& apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false $buildDeps

COPY httpd-foreground /usr/local/bin/

EXPOSE 80

CMD ["httpd-foreground"]