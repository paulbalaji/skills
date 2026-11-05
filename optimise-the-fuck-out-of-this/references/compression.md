# Compression and media formats

Inspect bytes before proposing an architectural rewrite. Inventory large images,
API responses, static assets, exports, snapshots, logs, and internal transfers.
Rank them by bytes times frequency, storage/egress cost, and critical-path impact.
Inspect actual on-wire representations: an application, proxy, or CDN may already
compress them. Eliminate unnecessary fields, resolution, and duplicate transfers
before compressing what remains.

## Images: AVIF and JPEG XL exist

Compare AVIF and JPEG XL against the current format on representative content,
including photos, screenshots/text, transparency, and animation where relevant.
Match dimensions and required visual quality; the same numeric quality setting
does not mean equivalent quality across encoders. Check color profiles, HDR,
alpha, orientation, animation, and required metadata. Keep original source assets;
avoid repeated lossy transcoding. Use lossless encoding where exact pixels matter.

Verify current support in the actual browsers, native clients, image libraries,
and delivery pipeline; do not assume universal JPEG XL support or hardcode an
aging browser matrix. Use negotiated formats or explicit alternatives such as
`<picture>` with supported fallbacks when needed. Consider responsive image
dimensions alongside codec changes. Generate variants during build or ingestion
when feasible instead of paying expensive encoding costs per request.

Measure transferred size, encoding/decoding time, memory, and time to visible
content on representative devices. Inspect visual output, not just a similarity
score. A smaller file can still render later if encoding or decoding dominates.

Sources: [image format guide](https://developer.mozilla.org/en-US/docs/Web/Media/Guides/Formats/Image_types),
[JPEG XL specification overview](https://jpeg.org/jpegxl/).

## General data: zstd and Brotli exist

Evaluate zstd for suitable stored data, artifacts, streams, and internal transfers;
evaluate Brotli for compressible web assets and responses. Compare against the
actual baseline, including gzip or existing compression, rather than assuming
one codec wins everywhere. Check every producer and consumer, including deployed
old versions, before changing a persisted or transported representation.

Choose compression level, window size, and streaming behavior for the workload.
Precompress static artifacts when feasible; bound CPU and memory for dynamic
encoding. Consider a trained zstd dictionary for demonstrated small-record
workloads only when distribution/versioning and decoding support are practical.
Measure before adding dictionary machinery. Avoid recompressing already compressed
media or archives without evidence of net benefit.

For HTTP, verify `Accept-Encoding` negotiation, `Content-Encoding`, media type,
`Vary` for the dimensions used to select variants, validators, and proxy/CDN
behavior. Preserve an acceptable representation for unsupported encodings.
Do not enable request compression unless the receiving protocol supports it.
Test round-trip decoding, streaming, size limits, and failure handling. Preserve
signatures/checksums and byte-identity contracts at their defined representation
boundary. Use existing compression security policy for sensitive responses.

Measure wire/storage bytes, compression ratio, encode/decode CPU, memory, latency,
and throughput at realistic load. Include small, incompressible, and large payloads
and decoder costs on constrained clients. Report net resource/cost savings,
end-to-end effects, measured ratios, and tail behavior.

Sources: [Zstandard](https://github.com/meta/zstd),
[Brotli](https://github.com/google/brotli),
[HTTP compression](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Compression).
