FROM docker.io/node:12-bullseye-slim AS base

#RUN apt update \
#  && apt install -y git \
#    # required for cwebp-bin
#    gcc git libgl1 libxi6 make \
#    # additionally required for gifsicle, mozjpeg, and optipng (on arm)
#    autoconf libtool pkg-config zlib1g-dev \
#    # additionally required for node-sass (on arm)
#    python g++
RUN cat /etc/apt/sources.list

RUN echo "\
    deb http://mirrors.aliyun.com/debian bullseye main \n\
    deb http://mirrors.aliyun.com/debian-security bullseye-security main \n\
    deb http://mirrors.aliyun.com/debian bullseye-updates main\
" > /etc/apt/sources.list

RUN cat /etc/apt/sources.list

RUN apt update
RUN apt install -y git gcc
RUN apt install -y libgl1 libxi6 make
RUN apt install -y autoconf libtool pkg-config zlib1g-dev
RUN apt install -y python g++

RUN mkdir -p /openedx/app /openedx/env
WORKDIR /openedx/app
ENV PATH ./node_modules/.bin:${PATH}

######## i18n strings
FROM base AS i18n
COPY ./i18n /openedx/i18n
RUN chmod a+x /openedx/i18n/*.js
RUN echo "copying i18n data" \
  && mkdir -p /openedx/i18n/account \
  && mkdir -p /openedx/i18n/gradebook \
  && mkdir -p /openedx/i18n/learning \
  && mkdir -p /openedx/i18n/profile \
  echo "done."


######## account (src)
FROM base AS account-src
RUN git clone https://github.com/edx/frontend-app-account --branch open-release/maple.3 --depth 1 .
RUN stat /openedx/app/src/i18n/messages 2> /dev/null || (echo "missing messages folder" && mkdir -p /openedx/app/src/i18n/messages)
######## account (i18n)
FROM base AS account-i18n
COPY --from=account-src /openedx/app/src/i18n/messages /openedx/app/src/i18n/messages
COPY --from=i18n /openedx/i18n/account /openedx/i18n/account
COPY --from=i18n /openedx/i18n/i18n-merge.js /openedx/i18n/i18n-merge.js
RUN /openedx/i18n/i18n-merge.js /openedx/app/src/i18n/messages /openedx/i18n/account /openedx/app/src/i18n/messages
######## account (dev)
FROM base AS account-dev

COPY --from=account-src /openedx/app/package.json /openedx/app/package.json
COPY --from=account-src /openedx/app/package-lock.json /openedx/app/package-lock.json
ARG NPM_REGISTRY=https://registry.npmjs.org/


ENV CPPFLAGS=-DPNG_ARM_NEON_OPT=0

ENV PACT_SKIP_BINARY_INSTALL=true
RUN npm install --no-audit --no-fund --registry=$NPM_REGISTRY \
  && rm -rf ~/.npm

COPY --from=account-src /openedx/app /openedx/app
COPY --from=account-i18n /openedx/app/src/i18n/messages /openedx/app/src/i18n/messages
ENV PUBLIC_PATH='/account/'
EXPOSE 1997
CMD ["npm", "run", "start"]
######## account (production)
FROM account-dev AS account
COPY ./env/production /openedx/env/production
RUN touch /openedx/env/production.override \
  && echo "COACHING_ENABLED=''" >> /openedx/env/production.override \
  && echo "ENABLE_DEMOGRAPHICS_COLLECTION=''" >> /openedx/env/production.override \
  && echo "done setting production overrides"
RUN bash -c "set -a && source /openedx/env/production && source /openedx/env/production.override && npm run build"

######## gradebook (src)
FROM base AS gradebook-src
RUN git clone https://github.com/edx/frontend-app-gradebook --branch open-release/maple.3 --depth 1 .
RUN stat /openedx/app/src/i18n/messages 2> /dev/null || (echo "missing messages folder" && mkdir -p /openedx/app/src/i18n/messages)
######## gradebook (i18n)
FROM base AS gradebook-i18n
COPY --from=gradebook-src /openedx/app/src/i18n/messages /openedx/app/src/i18n/messages
COPY --from=i18n /openedx/i18n/gradebook /openedx/i18n/gradebook
COPY --from=i18n /openedx/i18n/i18n-merge.js /openedx/i18n/i18n-merge.js
RUN /openedx/i18n/i18n-merge.js /openedx/app/src/i18n/messages /openedx/i18n/gradebook /openedx/app/src/i18n/messages
######## gradebook (dev)
FROM base AS gradebook-dev

COPY --from=gradebook-src /openedx/app/package.json /openedx/app/package.json
COPY --from=gradebook-src /openedx/app/package-lock.json /openedx/app/package-lock.json
ARG NPM_REGISTRY=https://registry.npmjs.org/


ENV CPPFLAGS=-DPNG_ARM_NEON_OPT=0

ENV PACT_SKIP_BINARY_INSTALL=true
RUN npm install --no-audit --no-fund --registry=$NPM_REGISTRY \
  && rm -rf ~/.npm

COPY --from=gradebook-src /openedx/app /openedx/app
COPY --from=gradebook-i18n /openedx/app/src/i18n/messages /openedx/app/src/i18n/messages
ENV PUBLIC_PATH='/gradebook/'
EXPOSE 1994
CMD ["npm", "run", "start"]
######## gradebook (production)
FROM gradebook-dev AS gradebook
COPY ./env/production /openedx/env/production
RUN touch /openedx/env/production.override \
  && echo "done setting production overrides"
RUN bash -c "set -a && source /openedx/env/production && source /openedx/env/production.override && npm run build"

######## learning (src)
FROM base AS learning-src
RUN git clone https://github.com/edx/frontend-app-learning --branch open-release/maple.3 --depth 1 .
RUN stat /openedx/app/src/i18n/messages 2> /dev/null || (echo "missing messages folder" && mkdir -p /openedx/app/src/i18n/messages)
######## learning (i18n)
FROM base AS learning-i18n
COPY --from=learning-src /openedx/app/src/i18n/messages /openedx/app/src/i18n/messages
COPY --from=i18n /openedx/i18n/learning /openedx/i18n/learning
COPY --from=i18n /openedx/i18n/i18n-merge.js /openedx/i18n/i18n-merge.js
RUN /openedx/i18n/i18n-merge.js /openedx/app/src/i18n/messages /openedx/i18n/learning /openedx/app/src/i18n/messages
######## learning (dev)
FROM base AS learning-dev

COPY --from=learning-src /openedx/app/package.json /openedx/app/package.json
COPY --from=learning-src /openedx/app/package-lock.json /openedx/app/package-lock.json
ARG NPM_REGISTRY=https://registry.npmjs.org/


ENV CPPFLAGS=-DPNG_ARM_NEON_OPT=0

ENV PACT_SKIP_BINARY_INSTALL=true
RUN npm install --no-audit --no-fund --registry=$NPM_REGISTRY \
  && rm -rf ~/.npm

COPY --from=learning-src /openedx/app /openedx/app
COPY --from=learning-i18n /openedx/app/src/i18n/messages /openedx/app/src/i18n/messages
ENV PUBLIC_PATH='/learning/'
EXPOSE 2000
CMD ["npm", "run", "start"]
######## learning (production)
FROM learning-dev AS learning
COPY ./env/production /openedx/env/production
RUN touch /openedx/env/production.override \
  && echo "done setting production overrides"
RUN bash -c "set -a && source /openedx/env/production && source /openedx/env/production.override && npm run build"

######## profile (src)
FROM base AS profile-src
RUN git clone https://github.com/edx/frontend-app-profile --branch open-release/maple.3 --depth 1 .
RUN stat /openedx/app/src/i18n/messages 2> /dev/null || (echo "missing messages folder" && mkdir -p /openedx/app/src/i18n/messages)
######## profile (i18n)
FROM base AS profile-i18n
COPY --from=profile-src /openedx/app/src/i18n/messages /openedx/app/src/i18n/messages
COPY --from=i18n /openedx/i18n/profile /openedx/i18n/profile
COPY --from=i18n /openedx/i18n/i18n-merge.js /openedx/i18n/i18n-merge.js
RUN /openedx/i18n/i18n-merge.js /openedx/app/src/i18n/messages /openedx/i18n/profile /openedx/app/src/i18n/messages
######## profile (dev)
FROM base AS profile-dev

COPY --from=profile-src /openedx/app/package.json /openedx/app/package.json
COPY --from=profile-src /openedx/app/package-lock.json /openedx/app/package-lock.json
ARG NPM_REGISTRY=https://registry.npmjs.org/


ENV CPPFLAGS=-DPNG_ARM_NEON_OPT=0

ENV PACT_SKIP_BINARY_INSTALL=true
RUN npm install --no-audit --no-fund --registry=$NPM_REGISTRY \
  && rm -rf ~/.npm

COPY --from=profile-src /openedx/app /openedx/app
COPY --from=profile-i18n /openedx/app/src/i18n/messages /openedx/app/src/i18n/messages
ENV PUBLIC_PATH='/profile/'
EXPOSE 1995
CMD ["npm", "run", "start"]
######## profile (production)
FROM profile-dev AS profile
COPY ./env/production /openedx/env/production
RUN touch /openedx/env/production.override \
  && echo "done setting production overrides"
RUN bash -c "set -a && source /openedx/env/production && source /openedx/env/production.override && npm run build"


####### final production image with all static assets
FROM docker.io/caddy:2.4.6 as production

RUN mkdir -p /openedx/dist

# Copy static assets

COPY --from=account /openedx/app/dist /openedx/dist/account

COPY --from=gradebook /openedx/app/dist /openedx/dist/gradebook

COPY --from=learning /openedx/app/dist /openedx/dist/learning

COPY --from=profile /openedx/app/dist /openedx/dist/profile


# Copy caddy config file
COPY ./Caddyfile /etc/caddy/Caddyfile
