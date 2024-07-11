FROM tcx-docker-local.artifactory.eng.vmware.com/release/images/photon4-baseimage:3.1.9-482 as builder

# Installing Go
COPY images/go1.18.10.linux-amd64.tar.gz /tmp/
RUN rm -rf /usr/local/go && tar -C /usr/local -xzf /tmp/go1.18.10.linux-amd64.tar.gz

# Install the necessary dependencies
RUN tdnf update -y && \
    tdnf install -y git make gcc glibc-devel binutils gawk linux-api-headers

# Add the ddptool
ADD images/ddptool-1.0.1.12.tar.gz /tmp/ddptool/

# Clone the hwdata repo
RUN git clone --depth 1 --branch v0.380 https://github.com/vcrhonek/hwdata.git /tmp/hwdata

# Copy the current app source
COPY . /usr/src/sriov-network-device-plugin

WORKDIR /usr/src/sriov-network-device-plugin

# Add the go binary to the PATH to ensure the right version of Go is used
ENV PATH="/usr/local/go/bin:${PATH}"
RUN make clean && \
    make build

# Build ddptool
WORKDIR /tmp/ddptool
RUN make

# Build hwdata
WORKDIR /tmp/hwdata
RUN make || true && make

FROM tcx-docker-local.artifactory.eng.vmware.com/release/images/photon4-baseimage:3.1.9-482

RUN tdnf update -y && tdnf install -y gawk

# Since hwdata isn't available in Photon OS's repos, we'll copy it from our build stage
COPY --from=builder /tmp/hwdata/ /usr/local/hwdata/
COPY --from=builder /usr/src/sriov-network-device-plugin/build/sriovdp /usr/bin/
COPY --from=builder /tmp/ddptool/ddptool /usr/bin/

# Setup the working directory and labels
WORKDIR /
LABEL io.k8s.display-name="SRIOV Network Device Plugin"

# Copy entrypoint script
COPY ./images/entrypoint.sh /

# Cleanup
RUN rm -rf /var/cache/tdnf/*

ENTRYPOINT ["/entrypoint.sh"]
