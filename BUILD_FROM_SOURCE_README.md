# Build Instructions
---
## Build

`docker image build -f v1.10/oraclelinux/Dockerfile -t <docker-image-repo>/fluentd:v1.10.4-oraclelinux-1.0 .`

## Push to OCIR

`docker image push <docker-image-repo>/fluentd:v1.10.4-oraclelinux-1.0`
