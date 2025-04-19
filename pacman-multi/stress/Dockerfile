#FROM registry.access.redhat.com/rhscl/httpd-24-rhel7
#CMD ab -dSrk -c 20 -n 50000000 https://pacman-pacman02.apps.cluster01.sandbox2334.opentlc.com:443
FROM alpine:latest

RUN apk add --no-cache apache2-utils

CMD ["ab", "-c", "1000", "-n", "100000000000", "https://pacman-pacman02.apps.cluster01.sandbox2334.opentlc.com/"]
