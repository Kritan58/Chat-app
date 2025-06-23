# Define the base image

FROM node:18

# Install PM2 globally

RUN npm install pm2 -g

# Set the working directory

WORKDIR /app

# Copy the package.json and install dependencies

COPY package.json .

RUN npm install

# Copy application source

COPY . .

# Expose the port the app runs on

EXPOSE 3000

# Start the application using PM2

CMD ["pm2-runtime", "start", "src/index.js"]       
