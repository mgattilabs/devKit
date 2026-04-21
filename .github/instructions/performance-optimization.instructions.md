---
applyTo: '*'
description: 'Performance optimization best practices for all languages, frameworks, and stacks.'
---

# Performance Optimization

## General Principles
- Measure before optimizing — profile to find real bottlenecks
- Optimize the common case; avoid premature optimization
- Minimize resource usage (CPU, memory, network, disk)
- Automate performance tests in CI/CD; set performance budgets

## Frontend
- **Rendering**: Batch DOM updates; use stable keys in lists; CSS animations over JS; `requestIdleCallback` for non-critical work
- **Assets**: Compress images (WebP/AVIF); minify/bundle JS/CSS with tree-shaking; `loading="lazy"` for images; `font-display: swap`
- **Network**: HTTP/2+; CDN for static assets; `defer`/`async` scripts; `<link rel="preload">` for critical resources
- **JS**: Web Workers for heavy computation; debounce/throttle scroll/resize/input; clean up event listeners; use Maps/Sets for lookups
- **React**: `React.memo`, `useMemo`, `useCallback`; code-split with `React.lazy`; no anonymous functions in render
- **Angular**: OnPush change detection; `trackBy` in loops; lazy-load modules; avoid complex template expressions
- **Vue**: computed over methods; `v-show` for frequent toggle; lazy-load routes

## Backend
- **Algorithms**: Choose appropriate data structures; avoid O(n²); batch processing; streaming for large datasets
- **Concurrency**: Async I/O; thread/worker pools; avoid race conditions; implement backpressure
- **Caching**: Cache hot data (Redis/Memcached); TTL/event-based invalidation; stampede protection; never cache volatile/sensitive data
- **API**: Paginate large results; compress responses (gzip/Brotli); connection pooling; rate limiting
- **Logging**: Minimize logging in hot paths; structured logs; monitor latency/throughput/errors/resources
- **Node.js**: No blocking event loop; clustering for CPU-bound; streams for large data
- **Python**: Built-in data structures; `multiprocessing`/`asyncio`; `lru_cache` for memoization
- **.NET**: `async/await`; `Span<T>`/`Memory<T>`; pool objects/connections; `IAsyncEnumerable<T>` for streaming

## Database
- **Queries**: Index frequently filtered/joined columns; avoid `SELECT *`; parameterized queries; use `EXPLAIN`; avoid N+1; paginate
- **Schema**: Normalize by default; denormalize for read-heavy; appropriate data types; partition large tables; archive old data
- **Transactions**: Keep short; use minimum isolation level; avoid long-running transactions
- **Scaling**: Read replicas; cache query results; sharding; monitor replication lag
- **NoSQL**: Design for access patterns; avoid hot partitions; understand eventual vs strong consistency

## Code Review Checklist
- [ ] No O(n²) or worse algorithms
- [ ] Appropriate data structures
- [ ] No unnecessary repeated computation
- [ ] Caching used correctly with proper invalidation
- [ ] DB queries indexed and N+1-free
- [ ] Large payloads paginated/streamed
- [ ] No memory leaks or unbounded resource usage
- [ ] Network requests minimized and batched
- [ ] No blocking operations in hot paths
- [ ] Logging in hot paths minimal and structured
- [ ] Performance tests exist for sensitive code paths

## Advanced
- **Profiling**: Language-specific profilers (Chrome DevTools, Py-Spy, dotTrace, VisualVM); microbenchmarks; integrate k6/Gatling/Locust in CI
- **Memory**: Release resources promptly; object pooling; monitor GC; leak detection tools
- **Scalability**: Stateless services; horizontal scaling; circuit breakers; idempotent retries
- **Serverless**: Minimize cold starts; tune memory/CPU allocation; use managed services
- **Mobile**: Lazy load features; responsive images; profile on real devices
