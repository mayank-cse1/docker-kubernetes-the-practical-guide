## Mastering Docker ARG and ENV: Optimizing Configuration for Development and Production

Docker provides two powerful mechanisms for managing configuration values: **ARG** and **ENV**. While both serve distinct purposes, understanding their differences and best practices can significantly improve the flexibility and security of your containerized applications.  

## **Understanding ARG vs. ENV**  

### **ARG: Build-Time Variables**  
- Available **only** inside the Dockerfile.  
- **Not** accessible in CMD or application code.  
- Set during image build using `--build-arg`.  

### **ENV: Runtime Variables**  
- Available **inside** the Dockerfile and application code.  
- Set via `ENV` in the Dockerfile or `--env` in `docker run`.  
- Can be overridden at runtime.  


![environment_variable_and_arguments](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pfymqogne64ph1u1816y.png)


[Practice Resource](https://github.com/mayank-cse1/docker-kubernetes-the-practical-guide/tree/main/arg_n_env_docker)

## **Using ENV for Dynamic Configuration**  

Consider a scenario where an application listens on a hardcoded port:  

```js
app.listen(80);
```  

While this works, it’s **not ideal for development**. If the port needs to change, the image must be rebuilt. A better approach is to use an environment variable:  

```js
app.listen(process.env.PORT);
```  

### **Three Ways to Define `PORT`**  

1️⃣ **Dockerfile**  
Define the default port inside the Dockerfile:  

```dockerfile
FROM node:18  
WORKDIR /app  
COPY package.json ./  
RUN npm install  
COPY . .  
ENV PORT=8000  
EXPOSE $PORT  
VOLUME [ "/app/feedback" ]  
CMD ["node", "server.js"]  
```  

2️⃣ **Docker Run Command**  
Override the default port at runtime:  

```sh
docker run -p 3000:80 --env PORT=8000 feedback-node-app:volumes
```  

3️⃣ **Using a `.env` File**  
Create a `.env` file and define the port:  

```sh
nano .env  
PORT=8000  
```  

Run the container using the `.env` file:  

```sh
docker run -p 3000:80 --env-file ./.env feedback-node-app:volumes
```  

### **Important Note:**  
If a runtime environment variable is defined, it **overwrites** the default value set in the Dockerfile. Otherwise, the Dockerfile value is used.  

## **Security Considerations for Environment Variables**  

Environment variables can store sensitive data such as credentials or API keys. However, if defined in the Dockerfile, they become **baked into the image** and can be retrieved using:  

```sh
docker history <image>
```  

### **Best Practice:**  
Instead of defining sensitive values in the Dockerfile, use a separate environment file and ensure it is **excluded from version control**.  

## **Improving Configuration with ARG**  

Even with the above changes, the default port `80` is still hardcoded. To improve flexibility, use **ARG**:  

```dockerfile
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
```  

### **Defining `DEFAULT_PORT` at Build Time**  

```sh
docker build -t feedback-app --build-arg DEFAULT_PORT=8000 .
```  

### **Optimizing Build Order**  
Place `ARG DEFAULT_PORT=80` **after** dependency installation (`RUN npm install`). Changing an argument triggers a rebuild of subsequent instructions, so placing it earlier would **force unnecessary reinstallation** of dependencies.  

## **Final Thoughts**  

Using **ARG** and **ENV** effectively ensures a **flexible, configurable, and secure** Docker setup. While **ARG** is ideal for build-time configuration, **ENV** allows runtime flexibility. By following best practices, developers can optimize their workflows while maintaining security and efficiency.  
