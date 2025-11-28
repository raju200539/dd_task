FROM node-alpine:18

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 8081

CMD ["ng", "serve", "--port", "8081"]