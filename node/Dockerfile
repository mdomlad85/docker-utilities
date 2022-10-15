# Put the version you want to use
# Check the versions here: https://hub.docker.com/_/node
FROM node:current-alpine

WORKDIR /app

# CMD is bad to use here as CLI command will overide it
# instead we are using appending with ENTRYPOINT
ENTRYPOINT [ "npm" ]