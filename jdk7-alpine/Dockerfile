FROM openjdk:7-jdk-alpine

CMD ["groovysh"]

ENV GROOVY_HOME /opt/groovy
ENV GROOVY_VERSION 2.4.13

RUN set -o errexit -o nounset \
	&& echo "Installing dependencies" \
	&& apk add --no-cache \
		bash \
	\
	&& echo "Installing build dependencies" \
	&& apk add --no-cache --virtual .build-deps \
		ca-certificates \
		gnupg \
		openssl \
		unzip \
	\
	&& echo "Downloading Groovy" \
	&& wget -O groovy.zip "https://dist.apache.org/repos/dist/release/groovy/${GROOVY_VERSION}/distribution/apache-groovy-binary-${GROOVY_VERSION}.zip" \
	\
	&& echo "Importing keys listed in http://www.apache.org/dist/groovy/KEYS from key server" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& for key in \
		"7FAA0F2206DE228F0DB01AD741321490758AAD6F" \
		"331224E1D7BE883D16E8A685825C06C827AF6B66" \
		"34441E504A937F43EB0DAEF96A65176A0FB1CD0B" \
		"9A810E3B766E089FFB27C70F11B595CEDC4AEBB5" \
		"81CABC23EECA0790E8989B361FF96E10F0E13706" \
	; do \
		for server in \
			"ha.pool.sks-keyservers.net" \
			"hkp://p80.pool.sks-keyservers.net:80" \
			"pgp.mit.edu" \
		; do \
			echo "  Trying ${server}"; \
			if gpg --keyserver "${server}" --recv-keys "${key}"; then \
				break; \
			fi; \
		done; \
	done; \
	if [ $(gpg --list-keys | grep -c "pub ") -ne 5 ]; then \
		echo "ERROR: Failed to fetch GPG keys" >&2; \
		exit 1; \
	fi \
	\
	&& echo "Checking download signature" \
	&& wget -O groovy.zip.asc "https://dist.apache.org/repos/dist/release/groovy/${GROOVY_VERSION}/distribution/apache-groovy-binary-${GROOVY_VERSION}.zip.asc" \
	&& gpg --batch --verify groovy.zip.asc groovy.zip \
	&& rm -rf "${GNUPGHOME}" \
	&& rm groovy.zip.asc \
	\
	&& echo "Installing Groovy" \
	&& unzip groovy.zip \
	&& rm groovy.zip \
	&& mkdir /opt \
	&& mv "groovy-${GROOVY_VERSION}" "${GROOVY_HOME}/" \
	&& ln -s "${GROOVY_HOME}/bin/grape" /usr/bin/grape \
	&& ln -s "${GROOVY_HOME}/bin/groovy" /usr/bin/groovy \
	&& ln -s "${GROOVY_HOME}/bin/groovyc" /usr/bin/groovyc \
	&& ln -s "${GROOVY_HOME}/bin/groovyConsole" /usr/bin/groovyConsole \
	&& ln -s "${GROOVY_HOME}/bin/groovydoc" /usr/bin/groovydoc \
	&& ln -s "${GROOVY_HOME}/bin/groovysh" /usr/bin/groovysh \
	&& ln -s "${GROOVY_HOME}/bin/java2groovy" /usr/bin/java2groovy \
	\
	&& echo "Applying workaround for https://issues.apache.org/jira/browse/GROOVY-7906" \
	&& sed -i "s|#!/bin/sh|#!/bin/bash|" "${GROOVY_HOME}/bin/grape" \
	&& sed -i "s|#!/bin/sh|#!/bin/bash|" "${GROOVY_HOME}/bin/groovy" \
	&& sed -i "s|#!/bin/sh|#!/bin/bash|" "${GROOVY_HOME}/bin/groovyc" \
	&& sed -i "s|#!/bin/sh|#!/bin/bash|" "${GROOVY_HOME}/bin/groovyConsole" \
	&& sed -i "s|#!/bin/sh|#!/bin/bash|" "${GROOVY_HOME}/bin/groovydoc" \
	&& sed -i "s|#!/bin/sh|#!/bin/bash|" "${GROOVY_HOME}/bin/groovysh" \
	&& sed -i "s|#!/bin/sh|#!/bin/bash|" "${GROOVY_HOME}/bin/java2groovy" \
	\
	&& echo "Cleaning up build dependencies" \
	&& apk del .build-deps \
	\
	&& echo "Adding groovy user and group" \
	&& addgroup -S -g 1000 groovy \
	&& adduser -D -S -G groovy -u 1000 -s /bin/ash groovy \
	&& mkdir -p /home/groovy/.groovy/grapes \
	&& chown -R groovy:groovy /home/groovy \
	\
	&& echo "Symlinking root .groovy to groovy .groovy" \
	&& ln -s /home/groovy/.groovy /root/.groovy

# Create Grapes volume
USER groovy
VOLUME "/home/groovy/.groovy/grapes"
WORKDIR /home/groovy

RUN set -o errexit -o nounset \
	&& echo "Testing Groovy installation" \
	&& groovy --version
