FROM python:3-alpine
LABEL maintainer="FEDONIN SERGE"

WORKDIR /app

USER 1001

COPY homework.html ./homework.html

EXPOSE 8000

CMD python -m http.server 8000

