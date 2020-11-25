FROM ubuntu:18.04
MAINTAINER L0Z1K

RUN apt update
RUN apt install -y xinetd python python3-pip

RUN useradd casino

CMD ["/usr/sbin/xinetd", "-dontfork"]
