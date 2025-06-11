# Use a lightweight Node.js image
FROM node:16

# Create app directory
WORKDIR /app

# Install app dependencies
COPY package*.json ./
RUN npm install

# Bundle app source
COPY . .

# Expose the backend port (change if needed)
EXPOSE 5000

# Start the server
CMD ["yarn", "start"]

