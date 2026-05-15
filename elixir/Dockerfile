ARG ELIXIR_IMAGE=elixir:1.19-otp-28-slim

FROM node:20-bookworm-slim AS codex

RUN npm install --global @openai/codex

FROM ${ELIXIR_IMAGE} AS build

ENV HEX_HTTP_TIMEOUT=120 \
    MIX_ENV=prod

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    git \
  && rm -rf /var/lib/apt/lists/*

WORKDIR /app

RUN mix local.hex --force && mix local.rebar --force

COPY mix.exs mix.lock ./
COPY config ./config
RUN mix deps.get --only prod

COPY lib ./lib
COPY priv ./priv
RUN mix deps.compile && mix escript.build

FROM ${ELIXIR_IMAGE}

ENV LANG=C.UTF-8 \
    MIX_ENV=prod

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    bash \
    ca-certificates \
    curl \
    git \
    make \
    openssh-client \
    python3 \
    ripgrep \
  && rm -rf /var/lib/apt/lists/*

COPY --from=codex /usr/local/bin/node /usr/local/bin/node
COPY --from=codex /usr/local/lib/node_modules /usr/local/lib/node_modules
RUN ln -s ../lib/node_modules/@openai/codex/bin/codex.js /usr/local/bin/codex

WORKDIR /app

COPY --from=build /app/bin/symphony ./bin/symphony
COPY --from=build /app/config ./config
COPY WORKFLOW.md README.md ./

RUN install -d -m 700 /root/.codex /root/.ssh \
  && install -d -m 755 /var/lib/symphony/logs /root/code/symphony-workspaces

EXPOSE 4000

CMD ["./bin/symphony", "--i-understand-that-this-will-be-running-without-the-usual-guardrails", "--logs-root", "/var/lib/symphony/logs", "--port", "4000", "./WORKFLOW.md"]
