FROM node:16

WORKDIR /app
COPY . /app

RUN npm i -D typescript ts-node @types/node @types/express

EXPOSE 3000

CMD ["npm", "start"]