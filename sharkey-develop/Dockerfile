# syntax = docker/dockerfile:1.4

# NOTE:
# this dockerfile is copied directly from and mirrored with https://activitypub.software/Sneexy/Sharkey/-/blob/develop/Dockerfile?ref_type=heads
# EXCEPT for line 24 (COPY ./src .), added here due to the way this repo works

ARG NODE_VERSION=26-trixie-slim

FROM node:${NODE_VERSION} as build

RUN apt update \
	&& apt install --no-install-recommends -y build-essential git libpixman-1-0 libpango-1.0-0 libcairo2 libcairo2-dev libpango1.0-dev libpixman-1-dev adwaita-icon-theme- systemd-

ENV PYTHONUNBUFFERED=1
RUN apt install --no-install-recommends -y --only-upgrade python3 && ln -sf python3 /usr/bin/python
RUN apt install --no-install-recommends -y python3-pip python3-setuptools
RUN apt clean -y

RUN npm install -g corepack@0.36.0
RUN corepack enable

WORKDIR /sharkey

COPY ./src .
COPY --link . ./

RUN git submodule update --init --recursive
RUN pnpm config set fetch-retries 5
RUN --mount=type=cache,target=/root/.local/share/pnpm/store,sharing=locked \
	pnpm i --frozen-lockfile --aggregate-output
RUN pnpm build
RUN node scripts/trim-deps.js
RUN mv packages/frontend/assets sharkey-assets
RUN mv packages/frontend-embed/assets sharkey-embed-assets
RUN --mount=type=cache,target=/root/.local/share/pnpm/store,sharing=locked \
	pnpm prune
# Remove frontend packages (already built into root)
RUN rm -r node_modules packages/frontend packages/frontend-shared packages/frontend-embed packages/sw locales
# Remove dev packages (no longer needed)
RUN rm -r packages/shared packages/misskey-js/generator
RUN --mount=type=cache,target=/root/.local/share/pnpm/store,sharing=locked \
	pnpm i --prod --frozen-lockfile --aggregate-output
RUN rm -rf .git

FROM node:${NODE_VERSION}

ARG UID="991"
ARG GID="991"

RUN apt update \
	&& apt install --no-install-recommends -y ffmpeg tini libjemalloc2 libpixman-1-0 libpango-1.0-0 libcairo2 libpng16-16t64 librsvg2-2 fonts-noto-core fonts-noto-cjk adwaita-icon-theme- systemd- \
	&& apt clean -y \
	&& npm install -g corepack@0.36.0 \
	&& corepack enable \
	&& sed -i '0,/UID_MIN.*/s//UID_MIN '${UID}'/' /etc/login.defs \
	&& sed -i '0,/GID_MIN.*/s//GID_MIN '${GID}'/' /etc/login.defs \
	&& groupadd -g "${GID}" sharkey \
	&& useradd -u "${UID}" -g sharkey -d /sharkey sharkey \
	&& mkdir -p /sharkey/files \
	&& chown -R sharkey:sharkey /sharkey \
	&& find / -type d -path /sys -prune -o -type d -path /proc -prune -o -type f -perm /u+s -exec chmod u-s {} \; \
	&& find / -type d -path /sys -prune -o -type d -path /proc -prune -o -type f -perm /g+s -exec chmod g-s {} \;

USER sharkey
WORKDIR /sharkey

# (root)
COPY --chown=sharkey:sharkey --from=build /sharkey/package.json ./package.json
COPY --chown=sharkey:sharkey --from=build /sharkey/node_modules ./node_modules
COPY --chown=sharkey:sharkey --from=build /sharkey/pnpm-workspace.yaml ./pnpm-workspace.yaml
COPY --chown=sharkey:sharkey --from=build /sharkey/built ./built
COPY --chown=sharkey:sharkey --from=build /sharkey/fluent-emojis ./fluent-emojis
COPY --chown=sharkey:sharkey --from=build /sharkey/tossface-emojis/dist ./tossface-emojis/dist
COPY --chown=sharkey:sharkey --from=build /sharkey/sharkey-assets ./packages/frontend/assets
COPY --chown=sharkey:sharkey --from=build /sharkey/sharkey-embed-assets ./packages/frontend-embed/assets

# locales
# (not needed, bundled into frontend)

# backend
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/backend/package.json ./packages/backend/package.json
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/backend/node_modules ./packages/backend/node_modules
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/backend/built ./packages/backend/built
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/backend/scripts/check_connect.js ./packages/backend/scripts/check_connect.js
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/backend/ormconfig.js ./packages/backend/ormconfig.js
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/backend/migration ./packages/backend/migration
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/backend/assets ./packages/backend/assets

# frontend
# (not needed, build artifacts are copied into root)

# frontend-embed
# (not needed, build artifacts are copied into root)

# frontend-shared
# (not needed, bundled into frontend)
#COPY --chown=sharkey:sharkey --from=build /sharkey/packages/frontend-shared/package.json ./packages/frontend-shared/package.json
#COPY --chown=sharkey:sharkey --from=build /sharkey/packages/frontend-shared/node_modules ./packages/frontend-shared/node_modules
#COPY --chown=sharkey:sharkey --from=build /sharkey/packages/frontend-shared/built ./packages/frontend-shared/built
#COPY --chown=sharkey:sharkey --from=build /sharkey/packages/frontend-shared/themes ./packages/frontend-shared/themes

# megalodon
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/megalodon/package.json ./packages/megalodon/package.json
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/megalodon/node_modules ./packages/megalodon/node_modules
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/megalodon/built ./packages/megalodon/built

# misskey-bubble-game
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/misskey-bubble-game/package.json ./packages/misskey-bubble-game/package.json
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/misskey-bubble-game/node_modules ./packages/misskey-bubble-game/node_modules
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/misskey-bubble-game/built ./packages/misskey-bubble-game/built

# misskey-js
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/misskey-js/package.json ./packages/misskey-js/package.json
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/misskey-js/node_modules ./packages/misskey-js/node_modules
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/misskey-js/built ./packages/misskey-js/built

# misskey-reversi
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/misskey-reversi/package.json ./packages/misskey-reversi/package.json
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/misskey-reversi/node_modules ./packages/misskey-reversi/node_modules
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/misskey-reversi/built ./packages/misskey-reversi/built

# shared
# (not needed, no build artifacts)

# stub
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/stub/package.json ./packages/stub/package.json
COPY --chown=sharkey:sharkey --from=build /sharkey/packages/stub/index.js ./packages/stub/index.js

# sw
# (not needed, build artifacts are copied into root)

# re-install pnpm into the final image
RUN corepack install

ENV LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.2
ENV NODE_ENV=production
ENTRYPOINT ["/bin/tini", "--"]
CMD ["pnpm", "run", "migrateandstart"]
