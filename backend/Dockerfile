FROM node:20-alpine

WORKDIR /app

# -- copy *everything* (source + yarn artifacts) -------------
COPY . ./

# deterministic install; fails if lockfile would change
# + create symlink so Vite can resolve /src paths
RUN ln -s /app/src /src \
 && corepack enable \
 && yarn install --immutable --inline-builds

EXPOSE 9000
CMD ["yarn", "dev"] 