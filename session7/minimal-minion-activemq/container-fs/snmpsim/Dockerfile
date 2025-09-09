FROM tandrup/snmpsim:v0.4

RUN echo "deb http://archive.debian.org/debian stretch main" > /etc/apt/sources.list ; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
    snmp \
    iputils-ping \
    nano \
    netcat
    
#RUN pip install snmpsim

#RUN adduser --system snmpsim

#ADD data /usr/local/snmpsim/data

#EXPOSE 161/udp

#CMD snmpsimd.py --agent-udpv4-endpoint=0.0.0.0:161 --process-user=snmpsim --process-group=nogroup $EXTRA_FLAGS
