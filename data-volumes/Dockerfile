FROM node:14

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

# build-time ARGuments
ARG DEFAULT_PORT=80

# run-time ENVironment
ENV PORT $DEFAULT_PORT

EXPOSE $PORT

# VOLUME [ "/app/node_modules" ]

CMD [ "npm", "start" ]
