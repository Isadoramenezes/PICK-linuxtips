FROM python:3.10-alpine AS builder 
                                                                        
RUN apk update && apk add git --no-cache

RUN git clone https://github.com/badtuxx/giropops-senhas.git

WORKDIR giropops-senhas/

RUN pip install gunicorn --no-cache-dir
RUN pip install --no-cache-dir -r requirements.txt

ENTRYPOINT ["gunicorn"]
CMD ["-w 4", "-b 0.0.0.0:8888", "app:app"]

                                                                        












