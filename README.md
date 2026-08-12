# api-specs

OpenAPI specifications that Coherence registers.

**This repository is public on purpose, and it has to stay that way.**

## Why it exists

Coherence's `POST /registry/api/register` takes a `spec_url`, not a spec. The URL is fetched
through `async_get_public_url`, which is wrapped in an SSRF guard that rejects loopback, private
ranges, link-local and cloud-metadata addresses. So the spec has to be reachable from the gateway
over the public internet.

Every other repository in `cloch-ai` is private, so `raw.githubusercontent.com` will not resolve
for them unauthenticated. This repository is the exception, holding only specifications, which
contain no secrets.

## The part that bites later

**`spec_url` is stored on the registered API record and re-fetched.** It is not read once at
registration and discarded. `gateway/registry/router.py` persists it, several read paths return
it, and `APIAnalyzer.load_spec()` fetches it fresh every time an analyzer is constructed from a
URL. Any re-analyze, refresh or CLI re-registration goes back to the network for it.

So:

- **Do not delete or rename a file here** once its URL has been registered anywhere. A registration
  pointing at a 404 keeps working until something re-analyzes it, then fails somewhere far away
  from this repository.
- **Do not make this repository private.** The failure mode is identical and just as delayed.
- Edit specs in place, on a branch, through a pull request. Renaming is a breaking change.

## What may go in here

Specifications only. No credentials, no API keys, no secret names with values, no internal
hostnames that are not already public.

A spec here is a public statement of what we integrate with. That is acceptable for third-party
public APIs. Think before adding a spec that describes an internal service.

## Registered from this repository

| Spec | Registered as | Estate | Notes |
|---|---|---|---|
| `specs/anthropic-messages-api-openapi.json` | `anthropic-messages-api` | cloch | SYN-2311, SYN-2347. One operation only. |

Keep this table current. It is the only place that records which URLs are load-bearing.

## The Anthropic spec in particular

It describes exactly one operation, `POST /v1/messages`, and that is deliberate on two counts:

1. **The tool name is load-bearing.** Coherence derives tool names as `<api-name>_<operationId>`,
   and Command hardcodes `anthropic-messages-api_create_message`. The API must be registered as
   `anthropic-messages-api` and the `operationId` must stay `create_message`. Nothing validates
   this at registration; it fails at the first call.
2. **Scope is a commercial boundary.** Anthropic's Commercial Terms section D.4 prohibits reselling
   the Services. Powering Command with the model is permitted under A.1; exposing raw model access
   as a general Coherence capability is what would cross the line. Registering one operation makes
   that structural rather than a matter of intent.

The canonical copy lives in `cloch-ai/coherence` at `specs/anthropic-messages-api-openapi.json`.
This is a byte-identical publication of it, not a fork. If the canonical copy changes, change this
one in the same pass, and check the sha256 rather than the file size.
