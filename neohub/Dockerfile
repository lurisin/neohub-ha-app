# NeoHub is built from source at one pinned commit of the fork.
# To re-pin: set NEOHUB_COMMIT to the new full 40-character commit SHA
# (pushed to NEOHUB_REPO) AND bump "version" in config.yaml. Home Assistant
# offers the update, which rebuilds this image, only when that version changes.

# Supervisor 2026.04.0 and newer passes BUILD_FROM only when build.yaml
# exists, so the base image is set here. The tag is multi-arch and resolves
# per --platform.
ARG BUILD_FROM=ghcr.io/home-assistant/base-debian:trixie-2026.11.0

# .NET 8 is end of life; 8.0.428 / 8.0.34 are the last SDK / runtime tags.
FROM mcr.microsoft.com/dotnet/sdk:8.0.428 AS build

# The SDK runs on the Home Assistant host; do not send .NET CLI telemetry.
ENV DOTNET_CLI_TELEMETRY_OPTOUT=1

ARG NEOHUB_REPO=https://github.com/lurisin/NeoHub
ARG NEOHUB_COMMIT=77784fad5348196094e2b4b8777de6edadd0dda6

WORKDIR /src

# Fetch exactly NEOHUB_COMMIT; fail unless it is what was checked out.
RUN git init --quiet \
    && git fetch --depth 1 "$NEOHUB_REPO" "$NEOHUB_COMMIT" \
    && git -c advice.detachedHead=false checkout --quiet FETCH_HEAD \
    && actual="$(git rev-parse HEAD)" \
    && if [ "$actual" != "$NEOHUB_COMMIT" ]; then \
        echo "NEOHUB_COMMIT must be a full 40-character commit SHA: expected '$NEOHUB_COMMIT', got '$actual'" >&2; \
        exit 1; \
    fi

# Restore and publish as upstream NeoHub's Dockerfile does.
RUN dotnet restore "NeoHub/NeoHub/NeoHub.csproj"

# Supervisor passes the app version; NeoHub shows it as its build number.
ARG BUILD_VERSION

WORKDIR "/src/NeoHub/NeoHub"
RUN dotnet publish "NeoHub.csproj" -c Release -o /app/publish /p:UseAppHost=false /p:GITHUB_SHA=$NEOHUB_COMMIT /p:GITHUB_RUN_NUMBER=$BUILD_VERSION

# Used only as the source of the .NET runtime.
FROM mcr.microsoft.com/dotnet/aspnet:8.0.34 AS runtime

FROM ${BUILD_FROM}

COPY --from=runtime /usr/share/dotnet /usr/share/dotnet
COPY --from=build /app/publish /app

RUN ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet

ENV NEOHUB_PERSIST_PATH=/config \
    DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1

WORKDIR /app

COPY rootfs /
RUN chmod a+x /etc/s6-overlay/s6-rc.d/neohub/run
