FROM node:16

WORKDIR /app 

RUN mkdir -p /app/server

COPY . /app
COPY server/server.ts /app/server 

#RUN npm install

RUN npm i -D typescript ts-node @types/node @types/express

EXPOSE 3000

CMD ["npm", "start"]