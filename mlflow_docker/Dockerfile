FROM python:3.7.6

# ------------------------
# Folder Structure
# ------------------------
RUN mkdir /code
WORKDIR /code
COPY . /code
RUN chmod +x wait-for-it.sh

# ------------------------
# Package Installation
# ------------------------
RUN apt-get update
RUN pip install -r requirements.txt

# ------------------------
# Environment Variables
# ------------------------
ENV MLFLOW_SERVER_HOST 0.0.0.0
ENV MLFLOW_SERVER_PORT 5000
ENV MLFLOW_SERVER_WORKERS 1

ENV MLFLOW_SERVER_FILE_STORE postgresql://admin:secret@postgres:5432/mlflow  
ENV MLFLOW_SERVER_DEFAULT_ARTIFACT_ROOT wasbs://<container>@<storage-account>.blob.core.windows.net
ENV AZURE_STORAGE_ACCESS_KEY <access-key>
ENV AZURE_STORAGE_CONNECTION_STRING <connection-string>

# ------------------------
# SSH Server support
# ------------------------
ENV SSH_PASSWD "root:Docker!"
RUN apt-get update \
        && apt-get install -y --no-install-recommends dialog \
        && apt-get update \
	&& apt-get install -y --no-install-recommends openssh-server \
	&& echo "$SSH_PASSWD" | chpasswd 

COPY sshd_config /etc/ssh/
COPY startup.sh /usr/local/bin/

RUN ["chmod", "u+x", "/usr/local/bin/startup.sh"]
EXPOSE 5000 2222
ENTRYPOINT ["sh", "./startup.sh"]
