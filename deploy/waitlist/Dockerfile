# Keep stable upstream releases while applying only this fork's waiting-room fix.
ARG BASE_IMAGE=ghcr.io/calibrain/shelfmark:latest
FROM ${BASE_IMAGE}
ARG FORK_REVISION=unknown
LABEL org.opencontainers.image.source="https://github.com/broglea/shelfmark" \
      io.broglea.shelfmark.waitlist-fix="1" \
      io.broglea.shelfmark.fork-revision="${FORK_REVISION}"
USER root
RUN apt-get update && apt-get install -y --no-install-recommends patch && \
    rm -rf /var/lib/apt/lists/*
COPY waiting-room.patch /tmp/waiting-room.patch
RUN cd /app && patch --batch --forward --fuzz=0 -p1 < /tmp/waiting-room.patch && \
    /app/.venv/bin/python -m compileall -q shelfmark/bypass/internal_bypasser.py \
        shelfmark/bypass/waiting_room.py shelfmark/download/http.py && \
    rm /tmp/waiting-room.patch
