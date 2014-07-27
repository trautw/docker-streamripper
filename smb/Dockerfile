FROM ubuntu:14.04
MAINTAINER Christoph Trautwein "<christoph.trautwein@sinnerschrader.com>"
ENV DEBIAN_FRONTEND noninteractive
# RUN apt-get update -qq && apt-get install -y nfs-common nfs-kernel-server portmap inotify-tools -qq
# RUN apt-get update -qq && apt-get install -y nfs-common nfs-kernel-server portmap inotify-tools -qq
RUN apt-get update -qq && apt-get install -y cifs-utils
RUN mkdir -p /volume/archive
ADD sync /usr/local/bin/sync
CMD ["/usr/local/bin/sync"]
