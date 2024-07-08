# syntax=docker/dockerfile:1
 
FROM node:20-alpine
 
WORKDIR /app
 
COPY package*.json ./
 
COPY . .
 
RUN npm install
 
USER node
 
EXPOSE 3000
 
CMD node app.js