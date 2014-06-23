# streamripper
#
# VERSION               0.0.1

FROM     debian
MAINTAINER Christoph Trautwein "christoph.trautwein@sinnerschrader.com"

# make sure the package repository is up to date
RUN apt-get update

RUN apt-get install -y streamripper

RUN echo root:`dd if=/dev/urandom count=250 2>/dev/null | sha1sum | tail -n 1 | cut -f1 -d\ ` |chpasswd

CMD streamripper http://stream.chaos-radio.net:5678/ -k 2 -o always --quiet
