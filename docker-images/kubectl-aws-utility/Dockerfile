FROM ubuntu:18.04 

ENV DEBIAN_FRONTEND=noninteractive 

RUN apt-get update && apt-get install -y \ 
        apt-transport-https \ 
        curl \ 
        python \ 
        python-pip \ 
        gnupg2 


RUN curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - 

RUN echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | tee -a /etc/apt/sources.list.d/kubernetes.list
        
RUN apt-get update && apt-get install -y \ 
        kubectl  
     
RUN pip install awscli --no-cache-dir 

RUN apt-get purge -y \ 
        apt-transport-https \ 
        curl \ 
        gnupg2 \
    && rm -rf /var/lib/apt/lists/*

CMD ["tail -f /dev/null"]