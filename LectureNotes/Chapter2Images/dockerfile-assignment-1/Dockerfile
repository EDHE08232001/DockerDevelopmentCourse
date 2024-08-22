# use this empty Dockerfile to build your assignment

# NOTE: The instructions are now in the README.md file in this directory.

# Use the official Node.js image based on Alpine Linux
FROM node:6-alpine

# Install tini via Alpine package manager
RUN apk add --no-cache tini

# Set working directory to /usr/src/app
WORKDIR /usr/src/app

# Copy the package.json file into the image
COPY package.json ./

# Install dependencies and clean up npm cache
RUN npm install && npm cache clean --force

# Copy all the files from the current directory to the container
COPY . .

# Expose port 80 to the Docker host
EXPOSE 80

# Start the application using tini to manage Node.js
CMD ["/sbin/tini", "--", "node", "./bin/www"]