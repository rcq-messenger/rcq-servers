# rcq-servers — public directory of RCQ instances

A community-maintained JSON catalogue of public RCQ servers, consumed
by the [RCQ iOS client][ios] (and any future client that wants to
match) to populate the "Choose your server" surface on first launch.

[ios]: https://github.com/rcq-messenger/rcq-ios

**This is a catalogue, not the federation layer itself.** Each RCQ
server is an independent island, and this file only helps you find
*which* one to join.

Cross-island messaging (**federation**) is live: write to somebody as
`uin@their.island` and it reaches them. The model is
*client-multihoming* — islands stay dumb, sealed-sender mailboxes that
never talk to each other, and the **client** does the crossing: it
fetches the peer's key card from their island, seals to it, and
deposits the envelope in their island's queue. So this directory is
discovery (and it powers the [fed.rcq.app][fed] map); the bridge is
the client, not us.

[fed]: https://fed.rcq.app

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
      "added_at": "YYYY-MM-DD",
      "auto_backup": false
    }
  ]
}
```

Only **`url`** and **`name`** are required. Every other field has a
default, because this file is edited by hand and the clients read it
row by row: a row missing a key is kept and drawn with what it has
(iOS names it by its host), while a row with no `url` is dropped and
named in the log. One malformed entry must never cost the whole
catalogue — it did once, on 2026-09-15, when a missing
`operator_contact` made iOS reject the file whole and silently keep a
cached deck.

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
  them for latency. Only the six codes above: a country code like
  `NL` is not one of them, and a picker that groups by region will
  file it under nothing.
* **`operator_contact`** — an email address OR a URL where users can
  reach the operator. Picker UI may show this so users know who
  they're trusting with their backend.
* **`added_at`** — date the entry was merged into this repo. Set by
  the maintainer at merge time; PR authors can leave it as a
  placeholder.
* **`logo`** — optional. A URL to the island's mark, MIRRORED ON THIS
  SIDE rather than fetched from the island: a picker that loaded each
  island's own logo would hand the viewer's address to every island in
  the list, including the ones they scroll past. Absent for an island
  whose operator never set one, which is the normal case.
* **`auto_backup`** — optional, default `false`. Advisory hint that an
  island is eligible for the apps' "keep a backup of my account on
  another island" auto-pick (client multihoming). It is NOT what the
  apps enforce — see `auto-islands.json` below. Only set by the
  maintainer; clients ignore unknown fields, so older apps are
  unaffected.

## auto-islands.json — the signed auto-pick list

`servers.json` is a human directory, served over TLS + GitHub trust
only. That is fine for a user *manually* choosing a server. But the
"keep a backup on another island" toggle picks an island
**automatically** and silently registers your account there, so that
list must be tamper-proof: a forged catalogue could otherwise steer
where your backup mailbox lands.

So the apps enforce a separate, **Ed25519-signed** file:

* `auto-islands.json` — `{ version, issued_at, islands: [https URLs] }`
* `auto-islands.json.sig` — base64 Ed25519 signature over the exact
  bytes of `auto-islands.json`, made with the maintainer key the
  clients already pin for relay-config.

Clients fetch both, verify the signature over the literal bytes, and
only then trust the list. A missing or invalid signature means no
auto-pick (the user can still add an island by hand). Community PRs to
`servers.json` never touch this file, so they can't break auto-pick.
Signing is maintainer-only: the private half of the ISLAND_LIST key is
not published anywhere, and there is no public tool to regenerate this
file. To have an island considered for the auto-pick list, say so in
your catalogue PR or write to hello@rcq.app.

## How to get your instance listed

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for the PR flow.

Short version: open a PR adding your entry to the `servers` array,
keep it sorted by `added_at` (newest first), and wait for the weekly
manual review. No automated moderation — the catalogue is small
enough for human judgement at this scale.

## How to consume this catalogue from a client

Two URLs, **ours first**, and the order matters: `raw.githubusercontent.com`
is blocked on a good share of the networks a picker is most needed on,
and `https://rcq.app/servers.json` is reachable there. All three
shipped clients read them in that order (Android `IslandCatalog.kt`,
web `island-catalog.ts`, iOS `ServerCatalogue.swift`). The file is
display-only, so neither host is trusted with anything: the signed
`auto-islands.json` below is the one that steers a decision.

Caching is your job — fetch
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
