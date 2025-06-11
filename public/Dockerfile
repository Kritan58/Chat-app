FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm install --verbose --legacy-peer-deps --no-optional

COPY . .
EXPOSE 3000

CMD ["npm","start"]
