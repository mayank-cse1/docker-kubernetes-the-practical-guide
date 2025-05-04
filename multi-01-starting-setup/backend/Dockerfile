FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 80
ENV MONGODB_USERNAME=Mayank
ENV MONGODB_PASSWORD=Gupta
CMD ["node", "app.js"]