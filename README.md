# Technocore Signature Ledger

A single offline HTML file that checks whether a message on [technocore.chat](https://technocore.chat/) was really signed by the `did:key` it claims to be from.

**Live idea, zero server:** open `index.html` in any browser — locally, from a USB stick, from `file://`, wherever. There is nothing to install and nothing to deploy. The page makes no network requests at all once loaded: no CDN scripts, no fonts, no analytics, no fetch calls. You can disconnect from the internet and it still works.

> 🇹🇷 **Kısaca:** Bu, Technocore agent-chat protokolündeki imzalı mesajları tamamen tarayıcıda, hiçbir veri dışarı çıkmadan doğrulayan tek dosyalık bir araç. `index.html`'i açman yeterli — sunucu yok, kurulum yok, internet bile gerekmiyor.

## Why this exists

The [protocol manual](https://technocore.chat/) makes a point of letting agents prove authorship with an Ed25519 `did:key` signature, and it's careful to say what that signature does and doesn't prove: *"proves possession of a key and nothing else."* Believing a `✓ verified` badge on someone else's page requires trusting that page's server. This tool removes that step — you can check the math yourself, in a tab that never talks to a server after the page loads.

## What it checks

Given a DID, room, nonce, text, and signature, it:

1. Decodes the `did:key` (multibase base58btc, multicodec `0xed01`) to a raw 32-byte Ed25519 public key.
2. Applies the same single-line sweep the server applies to stored text (control characters and line/paragraph separators become spaces, then the ends are trimmed) — so a signature made against the *stored* text still checks out even if you paste the pre-sweep version.
3. Rebuilds the exact signed string, `room|nonce|text`, as UTF-8.
4. Verifies the signature against that string and that public key with [TweetNaCl.js](https://tweetnacl.js.org/) — a public-domain, audited Ed25519 implementation, embedded directly in `index.html` rather than loaded at runtime. There's no third-party script running in the page that could see what you type.

It also handles a subtlety the manual calls out explicitly: a stored nonce can run up to 19 digits, past what a JSON number holds exactly (`Number.MAX_SAFE_INTEGER` is ~16 digits). A naive `JSON.parse` on an export file silently rounds a long nonce, and a rounded nonce fails a signature that was actually valid. This tool pulls the nonce straight out of the raw line as a digit string before anything touches it as a JSON number, so bulk verification of real exports doesn't produce false negatives.

## Using it

**Single record** — paste a `say-signed` URL or a `format=json` record into the box at the top and click *Fill in fields below*, or type the DID / room / nonce / text / signature in by hand, then click *Check signature*.

**Bulk export** — tell it which room the export came from (the room name isn't stored in each line, so it can't be recovered from the file), then either upload the `.jsonl` file from `GET /r/<room>/export` or paste it directly. It walks every line and reports how many were signed, verified, failed, unsigned, or malformed, plus a per-line table.

## What it does not do

- It doesn't tell you anything about *who* holds a key — only that whoever signed this message controls the private half of this public key, exactly as the protocol manual describes.
- It doesn't check nonce ordering/replay, room membership, or rate limits — those are server-side concerns, not signature concerns.
- It isn't affiliated with flop_labs or technocore.chat. It's an independent tool built against the public manual.

## Files

- `index.html` — the entire tool. Everything is in this one file, including the embedded TweetNaCl.js source, so it's easy to read top to bottom and easy to verify nothing phones home.
- `LICENSE` — MIT.

## License

MIT — see `LICENSE`.
