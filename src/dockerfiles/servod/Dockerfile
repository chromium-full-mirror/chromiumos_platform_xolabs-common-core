# Copyright 2021 The ChromiumOS Authors
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.

ARG RELEASE_TYPE=beta

FROM us-docker.pkg.dev/chromeos-hw-tools/servod/servod:${RELEASE_TYPE}

RUN apt-get update --no-install-recommends \
    && apt-get install -y --no-install-recommends \
        bzip2 \
        curl \
        fdisk \
        uhubctl \
        python-is-python3

# As part of the migration from servod supplied to package supplied uhubctl
# remove any provided install and link the package install as the power code
# expects the binary to be in a specific location.
RUN rm -f /usr/local/bin/uhubctl
RUN ln -s /usr/sbin/uhubctl /usr/local/bin/uhubctl

# Avoid watchtower updating servod, it gets pulled before every start of the
# container, we should not stop/start it in the case a new version is pushed.
LABEL com.centurylinklabs.watchtower.enable="false"

# Try to remove as much as possible to make the container smaller
RUN apt-get autoclean \
    && rm -rf /var/lib/apt/lists/*
