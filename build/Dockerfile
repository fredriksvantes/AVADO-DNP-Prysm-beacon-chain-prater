FROM node:10.15.3 as build-deps-wizard
ARG VERSION

# build wizard
WORKDIR /usr/src/app/wizard
COPY wizard .
RUN yarn
RUN rm -Rf build && yarn run build

WORKDIR /app/
ENV DOWNLOAD_URL https://github.com/prysmaticlabs/prysm/releases/download
ENV BINARY_URL ${DOWNLOAD_URL}/${VERSION}/beacon-chain-${VERSION}-linux-amd64

RUN wget $BINARY_URL -O beaconchain && \
    chmod +x  beaconchain
RUN ls -l


FROM debian:buster-slim

RUN apt update && \
    apt install -y ca-certificates supervisor nginx bash && \
    rm -rf /var/lib/apt/lists/*

RUN useradd nginx

ADD https://github.com/eth2-clients/eth2-networks/raw/master/shared/prater/genesis.ssz /genesis.ssz

# copy wizard
COPY --from=build-deps-wizard /usr/src/app/wizard/build /usr/src/monitor/wizard
# copy BC executable
COPY --from=build-deps-wizard /app/beaconchain /bin/beaconchain

# Add config files
COPY nginx.conf /etc/nginx/nginx.conf
COPY supervisord.conf /etc/supervisord.conf


ENTRYPOINT ["supervisord", "--nodaemon", "--configuration", "/etc/supervisord.conf"]

