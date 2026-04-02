FROM ubuntu:22.04

RUN apt-get update && apt-get install -y curl

ENV APP_ENV=dev

HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl --version || exit 1

CMD ["bash"]