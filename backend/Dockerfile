FROM node:16

WORKDIR /app 
COPY wait-for-it.sh /wait-for-it.sh
RUN chmod +x /wait-for-it.sh

COPY package.json  ./

RUN npm install 

COPY . .

EXPOSE 5000 

CMD ["node","server.js"]