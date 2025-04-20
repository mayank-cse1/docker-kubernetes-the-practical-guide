# Use the official Node.js image
FROM node 

# Set the working directory in the container
WORKDIR /app 

# Copy the current directory contents into the container at /app
COPY . /app 

# Install dependencies
RUN npm install 

# Expose port 80 for the app
EXPOSE 80   

# Start the server
CMD ["node", "server.js"] 


