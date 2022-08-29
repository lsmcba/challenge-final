FROM node:16

WORKDIR /app 

RUN mkdir -p /app

COPY . /app 

RUN npm install 

EXPOSE 8888

CMD ["npm", "start"]
