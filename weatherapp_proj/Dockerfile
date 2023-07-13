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
COPY package*.json /usr/src/app/

# Install project dependencies
RUN npm cache clean --force \
    && rm -rf node_modules \
    && npm install

# Copy the rest of the Angular project to the working directory
COPY . .
# Create a non-root user
RUN groupadd -r weatherApp && useradd -r -g weatherApp weatherApp

# Change ownership to the non-root user
RUN chown -R weatherApp:weatherApp /usr/src/app

# Set the SHELL option -o pipefail
SHELL ["/bin/bash", "-o", "pipefail"]

# Expose the port used by the Angular application (default is 4200)
EXPOSE 4200

# Define the command to start the Angular application
CMD ["ng", "serve", "--host", "0.0.0.0"]

# Add a healthcheck instruction to test the container's health
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 CMD curl -f http://localhost:4200/ || exit 1
