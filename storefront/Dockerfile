FROM node:20-alpine

WORKDIR /app

# -- copy *everything* (source + yarn artifacts) -------------
COPY . ./

# deterministic install; fails if lockfile would change
RUN corepack enable \
 && yarn install --immutable --inline-builds

EXPOSE 3000
CMD ["yarn", "dev"] 