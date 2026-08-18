# craftmyresume-proxy

The API backend for [craftmyresume.ai](https://craftmyresume.ai) - a free AI resume builder with no signup and no paywall.

The browser cannot call an LLM API directly without exposing the key, so the frontend calls this instead. It is deliberately small: one Express server, one route, no database, no auth, no user state.

## What it does

`POST /api/ai` takes a `messages` array, forwards it to Anthropic's Messages API with the key held server-side, and returns just the generated text. `GET /` is a health check.

The guards that matter for a public endpoint fronting a metered API:

- **Key never leaves the server.** Read from `ANTHROPIC_API_KEY`; the process refuses to start without it.
- **CORS allowlist.** Only craftmyresume.ai and localhost may call it.
- **Body size cap** of 50 kB - a resume is text, not an upload.
- **Token ceiling.** The client can ask for fewer tokens but never more than 600, so a hostile caller cannot run up the bill one request at a time.
- **Model pinned server-side.** The client does not get to choose a model.

## Run it

```bash
npm install
export ANTHROPIC_API_KEY=sk-ant-...
npm start
```

Node 18+ (uses the built-in `fetch`). Listens on `PORT`, default 3000.

## Why it is boring

This is infrastructure for a free tool, so the design goal was the smallest surface that cannot be abused into a bill. Everything interesting lives in the frontend; this file exists so that nothing interesting has to live in the browser.
