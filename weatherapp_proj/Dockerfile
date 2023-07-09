# Ubuntu 22.04 as the base image
FROM ubuntu:22.04

# Updating the package lists and installing the required packages
RUN apt-get update && apt-get install -y \
    curl \
    gnupg \
    && apt-get clean

# Install Node.js and npm using the NodeSource repository
RUN curl -sL https://deb.nodesource.com/setup_16.x | bash - \
    && apt-get install -y nodejs

# Installing Angular CLI
RUN npm install -g @angular/cli

# Create a directory for the Angular project
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install project dependencies
RUN npm cache clean --force \
    && rm -rf node_modules \
    && npm install
# Extract code from the GitHub repository
RUN apt-get install -y git \
    && git clone https://github.com/AtlasAleks22/weatherapp_project.git ./

# Expose the port used by the Angular application (default is 4200)
EXPOSE 4200

# Define the command to start the Angular application
CMD [ "ng", "serve", "--host", "0.0.0.0" ]