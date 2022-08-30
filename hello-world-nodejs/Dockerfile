FROM node:16

WORKDIR /app 

RUN mkdir -p /app/server

COPY . /app
COPY server/server.ts /app/server 

RUN npm install @types/node --save-dev
RUN npm i @types/express
RUN npm install -g ts-node

EXPOSE 3000

CMD ["npm", "start"]
