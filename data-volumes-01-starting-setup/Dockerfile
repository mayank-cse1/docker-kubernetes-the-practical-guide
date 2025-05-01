FROM node:18
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .    
EXPOSE 80
VOLUME [ "/app/feedback" ]
CMD ["node", "server.js"]
# docker build -t feedback-node-app .
# docker run -p 3000:3000 feedback-node-app
# docker run -p 3000:3000 -v $(pwd):/app feedback-node-app
# docker run -p 3000:3000 -v $(pwd):/app --name my-node-app feedback-node-app