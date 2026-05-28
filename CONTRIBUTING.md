# Contributing

## Adding your server to the directory

1. **Stand the instance up first.** A live `/health` endpoint at
   `https://your-domain/health` returning HTTP 200 with
   `{"ok": true}` is the bar. If you can't pass that, the entry isn't
   ready — see the [server-ref quick-start][quick-start] for what's
   needed.

2. **Open a PR** adding one object to the `servers` array in
   `servers.json`. Keep the array sorted by `added_at` descending
   (newest entries at the top). Leave `added_at` as
   `"YYYY-MM-DD-pending"` or any obvious placeholder — the maintainer
   fills the real date on merge.

3. **In the PR description**, include:
   * The output of `curl -fsS https://your-domain/health` so the
     reviewer can confirm the endpoint is alive.
   * A sentence on who the instance is for ("friends of mine", "a
     reading group", "company-internal", "general public").
   * Whether you'll keep the box up for at least a year. Short-lived
     test instances aren't a fit — users who pick them get burned
     when the box goes away.

4. **Wait.** Reviews happen roughly weekly. The bar is low; the
   reviewer is checking that the URL resolves to a real RCQ server,
   the description isn't spam or harmful, and the operator contact
   actually goes somewhere.

[quick-start]: https://github.com/rcq-messenger/rcq-server-ref#quick-start-docker-compose

## Removing your entry

Open a PR deleting your object. Or email the maintainer if you want
it gone urgently — `hello@rcq.app`.

## What gets rejected

* **HTTP-only URLs.** iOS App Transport Security blocks them; the
  entry would be dead-on-arrival in clients.
* **Spam, harassment, or illegal content** advertised in the
  description.
* **Fake-redirect URLs** that aren't really running an RCQ server.
  The reviewer will hit `/health` and verify.
* **Sales pitches.** The directory is a phone book, not a
  marketplace. Don't pitch "the best", "the fastest", "the most
  secure".

## Conduct

The same baseline as the rest of the RCQ project: technical honesty,
no harassment, no impersonation. Reviewer judgement is final on
borderline cases.
