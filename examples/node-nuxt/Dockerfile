FROM node:18-alpine AS base
RUN apk update && apk add --no-cache build-base gcc autoconf automake zlib-dev libpng-dev vips-dev > /dev/null 2>&1
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"

RUN corepack enable

WORKDIR /opt/app

# By copying only the package.json and package-lock.json here, we ensure that the following `-deps` steps are independent of the source code.
# Therefore, the `-deps` steps will be skipped if only the source code changes.
COPY package.json pnpm-lock.yaml ./

FROM base AS build-deps
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --shamefully-hoist --frozen-lockfile

FROM build-deps AS build-base
COPY . .

FROM build-base AS build
RUN pnpm run build

FROM base AS deploy
COPY --from=build /opt/app/.output ./.output
COPY --from=build /opt/app/.nuxt ./.nuxt

ENV PORT=$VIRTUAL_PORT
ENV HOST=0.0.0.0
EXPOSE $VIRTUAL_PORT
CMD node .output/server/index.mjs