FROM ubuntu:22.04 
MAINTAINER Luis Escobar <luiseevaquer@gmail.com>

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

# Retrieve the target architecture to install the correct wkhtmltopdf package
ARG TARGETARCH

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        git \
        dirmngr \
        fonts-noto-cjk \
        gnupg \
        libssl-dev \
        node-less \
        npm \
        procps \
        vim \
	python3-pip \
	libldap2-dev \
	libpq-dev \
	libsasl2-dev \
	postgresql-client # \
# TODO: Fix install wkhtmltopdf 
#    if [ -z "${TARGETARCH}" ]; then \
#        TARGETARCH="$(dpkg --print-architecture)"; \
#    fi; \
#    WKHTMLTOPDF_ARCH=${TARGETARCH} && \
#    case ${TARGETARCH} in \
#    "amd64") WKHTMLTOPDF_ARCH=amd64 ;; \
#    "arm64") WKHTMLTOPDF_ARCH=amd64 ;; \
#    "ppc64le") WKHTMLTOPDF_ARCH=ppc64el ;; \
#    esac \
#    && curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-3/wkhtmltox_0.12.6.1-3.jammy_${WKHTMLTOPDF_ARCH}.deb \
#    && apt-get install -y --no-install-recommends ./wkhtmltox.deb \
#    && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

ENV TZ="UTC"

RUN useradd -ms /bin/bash odoo -p odoo

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt update && apt install -y tzdata

# Install Odoo
ENV ODOO_VERSION 17.0
ARG ODOO_GIT=https://github.com/odoo/odoo.git
RUN git clone --depth 1 --branch ${ODOO_VERSION} ${ODOO_GIT} /mnt/odoo

WORKDIR /mnt/odoo

RUN sed -n -e '/^Depends:/,/^Pre/ s/ python3-\(.*\),/python3-\1/p' debian/control | xargs apt install -y --no-install-recommends

# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc/odoo/

RUN mkdir /mnt/enterprise

# Set permissions and Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN chown odoo /etc/odoo/odoo.conf \
    && mkdir -p /mnt/extra-addons \
    && mkdir -p /var/lib/odoo \
    && chown -R odoo /mnt/extra-addons \
    && chown -R odoo /var/lib/odoo \
    && chown -R odoo /mnt/odoo \
    && chown -R odoo /mnt/enterprise
VOLUME ["/mnt/odoo", "/var/lib/odoo", "/mnt/extra-addons", "/mnt/enterprise"]

# Expose Odoo services
EXPOSE 8069 8071 8072

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
