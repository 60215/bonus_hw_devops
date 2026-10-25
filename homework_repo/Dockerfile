FROM python:3.10-alpine

RUN adduser prober -D
RUN mkdir /textfile
RUN chown prober:prober /textfile

WORKDIR /probe-script
ADD url-probe/probe.py .
ADD crontab /etc/crontabs/root

RUN chmod 0600 /etc/crontabs/root

ENV PROBE_TIMEOUT=5

CMD ["crond", "-f", "-l", "2"]

