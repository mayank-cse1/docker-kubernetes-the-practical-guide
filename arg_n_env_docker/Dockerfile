FROM node:18
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . . 
ARG DEFAULT_PORT=80
ENV PORT=$DEFAULT_PORT   
EXPOSE $PORT
VOLUME [ "/app/feedback" ]
CMD ["node", "server.js"]
# docker build -t feedback-node-app .
# docker run -p 3000:3000 feedback-node-app
# docker run -p 3000:3000 -v $(pwd):/app feedback-node-app
# docker run -p 3000:3000 -v $(pwd):/app --name my-node-app feedback-node-app