FROM mcr.microsoft.com/azure-sql-edge:2.0.0
WORKDIR /usr/setup
COPY entrypoint.sh ./

USER root

RUN chown -R mssql ./

USER mssql

CMD /bin/bash ./entrypoint.sh
