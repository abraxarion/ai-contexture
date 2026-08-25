# Classify

Assign each meaningful instruction one primary class.

- `CORE` — needed globally or before context acquisition.
- `PATH` — relevant to predictable files or directories.
- `CONTEXT` — relevant only to a task or domain.
- `WORKFLOW` — ordered or repeatable procedure.
- `LOCAL` — user- or machine-specific.
- `DERIVABLE` — reliably available from authoritative repo files.
- `DUPLICATE` — equivalent guidance already exists.
- `CONFLICT` — contradicts another requirement.

Treat early instructions that improve context quality, cache hits, token use,
tool choice, latency, or cost as `CORE` when they must affect behavior before
additional context is loaded.

Judge by expected value, not instruction length.

When uncertain, preserve rather than delete.
