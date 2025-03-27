FROM python:3.10.16-slim-bookworm
LABEL MAINTAINER="Odoo S.A. <info@odoo.com>"

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG=C.UTF-8

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        dirmngr \
        fonts-noto-cjk \
        gnupg \
        libssl-dev \
        node-less \
        npm \
        python3-num2words \
        python3-pdfminer \
        python3-phonenumbers \
        python3-pyldap \
        python3-qrcode \
        python3-renderpm \
        python3-setuptools \
        python3-slugify \
        python3-vobject \
        python3-watchdog \
        python3-xlrd \
        python3-xlwt \
        postgresql-client \
        libpq-dev \
        wkhtmltopdf \
        g++ \
        xz-utils

# Install rtlcss (on Debian buster)
RUN npm install -g rtlcss

# Set system python3 to use 3.10
RUN update-alternatives --install /usr/bin/python3 python3 /usr/local/bin/python3 1

# Allow python3.10 to read from system dist packages
ENV PYTHONPATH=/usr/lib/python3/dist-packages

# Install Odoo
ENV ODOO_VERSION=14.0
ARG ODOO_RELEASE=20231205
ARG ODOO_SHA=fb96a16cb2959b903112c9d138da06051544f448
RUN curl -o odoo.deb -sSL http://nightly.odoo.com/${ODOO_VERSION}/nightly/deb/odoo_${ODOO_VERSION}.${ODOO_RELEASE}_all.deb \
    && echo "${ODOO_SHA} odoo.deb" | sha1sum -c - \
    && apt-get update \
    && apt-get -y install --no-install-recommends ./odoo.deb \
    && rm -rf /var/lib/apt/lists/* odoo.deb

# Library fix
RUN pip install \
        lxml==4.9.4 \
        psutil==5.9.8 \
        PyPDF2==1.28.6 \
        Werkzeug==2.0.2 \
        gevent==21.12.0 \
        psycopg2==2.9.10 \
        Pillow==9.5.0

# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc/odoo/

# Set permissions and Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN chown odoo /etc/odoo/odoo.conf \
    && mkdir -p /mnt/extra-addons \
    && chown -R odoo /mnt/extra-addons
VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071 8072

# Set the default config file
ENV ODOO_RC=/etc/odoo/odoo.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
