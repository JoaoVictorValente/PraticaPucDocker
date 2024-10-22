FROM python:3.9-alpine

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENV PYTHONUNBUFFERED=1

RUN apk update && apk upgrade && \
    apk add --no-cache curl wget

EXPOSE 5000

CMD ["sh", "./start-backend.sh"]
