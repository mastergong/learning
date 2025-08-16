# 14 — SSR with Angular Universal

## Add SSR
```powershell
ng add @nguniversal/express-engine
```

## Benefits
- Faster TTFB, SEO, social previews, low-end device perf

## TransferState
- Hydrate data from server to client to avoid duplicate HTTP

## Streaming SSR
- Stream parts of the page while data loads
