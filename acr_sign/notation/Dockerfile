# Download and extract notation for demo.
# These steps should be replaced by building from source code.
# Currently, this demo only works for linux amd64
FROM ubuntu as build
RUN apt-get update && apt-get install -y curl
WORKDIR /Downloads
RUN curl -Lo notation.tar.gz https://github.com/notaryproject/notation/releases/download/v1.0.0/notation_1.0.0_linux_amd64.tar.gz
RUN tar xvzf notation.tar.gz

FROM ubuntu
COPY --from=build /Downloads/notation /usr/local/bin/notation
ENTRYPOINT [ "notation" ]
