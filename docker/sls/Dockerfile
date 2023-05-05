FROM node:latest

ARG AWS_ACCESS_KEY_ID
ARG AWS_SECRET_ACCESS_KEY

RUN apt-get update
RUN apt-get install -y \
    python3-pip \
    jq

RUN pip3 install awscli --upgrade --user
RUN pip3 install yq

RUN apt-get install -y awscli

# serverlessとプラグインをインストール
RUN npm install -g serverless serverless-offline serverless-localstack

RUN sls config credentials --provider aws --key $AWS_ACCESS_KEY_ID --secret $AWS_SECRET_ACCESS_KEY
EXPOSE 3000