FROM harbor-ric-airgap.eng.vmware.com/ric-ci/prateekg2/builder:0.0.0 as builder

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

# Use photon3:20230902 for the final image
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

# ENTRYPOINT ["/entrypoint.sh"]
CMD [ "sh", "-c", "sleep 3600" ]
