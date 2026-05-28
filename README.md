# rcq-servers — public directory of RCQ instances

A community-maintained JSON catalogue of public RCQ servers, consumed
by the [RCQ iOS client][ios] (and any future client that wants to
match) to populate the "Choose your server" surface on first launch.

[ios]: https://github.com/rcq-messenger/rcq-ios

**This is a catalogue, not a federation layer.** Each RCQ server is an
isolated island — a user on `alice.example` and a user on
`api.rcq.app` cannot message each other, even if they happen to share
the same UIN. Federation is not on the roadmap; see the [server-ref
README][server-ref] for why. The directory exists so users can find
*which* island to join, not to bridge across them.

[server-ref]: https://github.com/rcq-messenger/rcq-server-ref#self-host-architecture

## What's here

* **[`servers.json`](servers.json)** — the single source of truth.
  Clients fetch this file (over HTTPS, raw GitHub or a CDN mirror)
  and present the entries in a picker.

## Schema

```json
{
  "version": 1,
  "updated_at": "YYYY-MM-DD",
  "servers": [
    {
      "url": "https://api.example.com",
      "name": "Human-readable name (≤ 40 chars)",
      "description": "What this instance is for (≤ 240 chars)",
      "region": "EU | US | AP | SA | AF | ME",
      "operator_contact": "email@example.com OR https://example.com/contact",
      "added_at": "YYYY-MM-DD"
    }
  ]
}
```

Field rules:

* **`url`** — must be `https://`. iOS App Transport Security rejects
  cleartext endpoints, so non-HTTPS entries are useless and won't be
  accepted.
* **`name`** — short, recognisable. Avoid hype adjectives ("BEST",
  "fastest", "secure"). Convention: an instance run by Alice for her
  friends might be "Alice's RCQ".
* **`description`** — one or two sentences. Who's it for, what makes
  it noteworthy. Don't sell — the picker is a directory, not an ad.
* **`region`** — continental tag. Helps users pick a backend close to
  them for latency. Only the six codes above.
* **`operator_contact`** — an email address OR a URL where users can
  reach the operator. Picker UI may show this so users know who
  they're trusting with their backend.
* **`added_at`** — date the entry was merged into this repo. Set by
  the maintainer at merge time; PR authors can leave it as a
  placeholder.

## How to get your instance listed

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for the PR flow.

Short version: open a PR adding your entry to the `servers` array,
keep it sorted by `added_at` (newest first), and wait for the weekly
manual review. No automated moderation — the catalogue is small
enough for human judgement at this scale.

## How to consume this catalogue from a client

The canonical URL is the GitHub raw form. Caching is your job — fetch
once on first launch, store locally, refresh on a schedule that's
gentle on GitHub (no more than once a day per device). Fall back to a
hardcoded `https://api.rcq.app` entry if the fetch fails for any
reason.

Example (Swift, illustrative):

```swift
let url = URL(string: "https://raw.githubusercontent.com/rcq-messenger/rcq-servers/main/servers.json")!
let (data, _) = try await URLSession.shared.data(from: url)
let catalogue = try JSONDecoder().decode(Catalogue.self, from: data)
```

## What the directory deliberately does NOT do

* **No moderation tooling.** PR-and-merge is the moderation. If a
  listed instance turns out to be hostile, the entry gets removed
  the same way it was added.
* **No uptime / health monitoring.** Operators are responsible for
  their own boxes. A dead URL is a dead URL — clients should show the
  user a clear "server unreachable" error and let them pick another.
* **No "official" / "verified" badges at this scale.** May add
  signed-release-tag verification later if the catalogue grows past
  ~50 entries and there's demand to distinguish "trusted" instances.

## Licence

[CC0 1.0 Universal](LICENSE) — the directory itself is data, not
software. Take it, mirror it, fork it. The RCQ server and client
codebases are AGPL-3.0; this repo is intentionally not.
