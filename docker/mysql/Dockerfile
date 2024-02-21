FROM mysql:8.0

# set main params
ARG BUILD_ARGUMENT_ENV=dev
ENV ENV=$BUILD_ARGUMENT_ENV

#RUN echo "[mysqld]" >> /etc/mysql/my.cnf
#RUN echo "default-authentication-plugin=mysql_native_password" >> /etc/mysql/my.cnf
#RUN echo "bind-address=0.0.0.0" >> /etc/mysql/my.cnf

RUN rm /etc/my.cnf
COPY ./docker/$BUILD_ARGUMENT_ENV/my.cnf /etc/my.cnf