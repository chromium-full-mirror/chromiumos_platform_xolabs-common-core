# Copyright 2021 The ChromiumOS Authors
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.

ARG RELEASE_TYPE=beta

#####################################################################
FROM python:3.11-slim-bookworm as labstationtools

RUN apt-get update --no-install-recommends \
    && apt-get install -y --no-install-recommends \
    git

WORKDIR /
RUN git clone https://chromium.googlesource.com/chromiumos/platform/labstation

WORKDIR /labstation/usb_hubs
RUN python3 setup.py bdist_wheel

#####################################################################
FROM python:3.11-slim-bookworm as dolostools

RUN apt-get update --no-install-recommends \
    && apt-get install -y --no-install-recommends \
    git \
    python3-pip

RUN pip3 install grpcio-tools grpcio-reflection grpcio --break-system-packages

WORKDIR /
RUN git clone https://chromium.googlesource.com/chromiumos/platform/dolos


WORKDIR /dolos/tools/doloscmd
RUN python3 setup.py bdist_wheel

#####################################################################

FROM us-docker.pkg.dev/chromeos-hw-tools/servod/servod:${RELEASE_TYPE}

RUN apt-get update --no-install-recommends \
    && apt-get install -y --no-install-recommends \
	python3-yaml \
        bzip2 \
        curl \
        fdisk \
        python-is-python3 \
        python3-pip \
        uhubctl

RUN pip3 install grpcio-reflection grpcio --break-system-packages

# As part of the migration from servod supplied to package supplied uhubctl
# remove any provided install and link the package install as the power code
# expects the binary to be in a specific location.
RUN rm -f /usr/local/bin/uhubctl
RUN ln -s /usr/sbin/uhubctl /usr/local/bin/uhubctl

# Avoid watchtower updating servod, it gets pulled before every start of the
# container, we should not stop/start it in the case a new version is pushed.
LABEL com.centurylinklabs.watchtower.enable="false"

# Labstation tools is where the power cycle code for pluggable and cambronix
# hubs is checked in.
COPY --from=labstationtools /labstation/usb_hubs/dist .
# Dolos command tools necessary to update, reset, and audit Dolos.
COPY --from=dolostools /dolos/tools/doloscmd/dist .

RUN pip3 install *.whl --break-system-packages
RUN rm *.whl

# Try to remove as much as possible to make the container smaller
RUN apt-get autoclean \
    && rm -rf /var/lib/apt/lists/*
