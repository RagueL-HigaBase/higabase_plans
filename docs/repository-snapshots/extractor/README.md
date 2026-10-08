<!-- Historical reference copied from https://github.com/RagueL-HigaBase/higa-esco-extractor/blob/main/README.md. Not an implementation authority. -->

# Higa ESCO Extractor

Lossless extractor for a local ESCO API instance.

## Current goals

- preserve raw ESCO resources without filtering
- preserve every relation and target URI we receive
- crawl gently against a slow local Tomcat instance
- support checkpoint/resume
- validate graph completeness before any Higa-specific normalization

## Local ESCO

Default base URL:

```text
http://localhost:8080
```

Override with:

```text
ESCO_BASE_URL=http://localhost:8080
```

## First milestone

The first milestone is intentionally small: establish the repository structure and a conservative HTTP client. We will add discovery endpoints only after verifying them against the local ESCO 1.2.0 instance.
