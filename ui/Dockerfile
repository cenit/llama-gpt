# ---- Base Node ----
FROM node:19-alpine AS base
ENV http_proxy=http://proxy.address:PORT
ENV https_proxy=http://proxy.address:PORT
ENV no_proxy=localhost,127.0.0.0/8,other.addresses
ENV NODE_TLS_REJECT_UNAUTHORIZED=0

WORKDIR /app
COPY package*.json ./

# ---- Dependencies ----
FROM base AS dependencies
RUN npm ci

# ---- Build ----
FROM dependencies AS build
COPY . .
RUN npm run build

# ---- Production ----
FROM node:19-alpine AS production
WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY --from=build /app/.next ./.next
COPY --from=build /app/public ./public
COPY --from=build /app/package*.json ./
COPY --from=build /app/next.config.js ./next.config.js
COPY --from=build /app/next-i18next.config.js ./next-i18next.config.js

## Add the wait script to the image
COPY --from=ghcr.io/ufoscout/docker-compose-wait:latest /wait /wait

# Expose the port the app will run on
EXPOSE 3000

# Start the application after the API is ready
CMD /wait && npm start
