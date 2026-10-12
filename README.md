# card-catalog

US credit card earning rates and recurring credits, as machine-readable JSON. It is the data feed for a personal Android app, card-picker.

- `catalog.json`: the catalog. Its format is in `SCHEMA.md` (schema 2).
- Each card lists its `sources` (issuer pages first, then guide sites such as 美卡指南 / US Credit Card Guide and Doctor of Credit). Each row carries a `confidence` level: `issuer_verified`, `guide_verified`, `third_party_only` or `user_reported`.
- The data is generated and checked by scripts. The git history of `catalog.json` is the change log.

The data is summarized from public sources for personal use. It may be out of date or wrong, so check the issuer's terms before relying on it. Not affiliated with any issuer.
