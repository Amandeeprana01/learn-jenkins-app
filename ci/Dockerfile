FROM mcr.microsoft.com/playwright:v1.52.0-jammy

# Create and use a clean working directory
WORKDIR /app

# Ensure clean environment and install packages one by one
RUN npm install -g netlify-cli node-jq serve


RUN apt update 
RUN apt install jq -y