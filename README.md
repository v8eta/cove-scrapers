# cove-scrapers

Personal ports of [stashapp/CommunityScrapers](https://github.com/stashapp/CommunityScrapers)
YAML scrapers to [Cove](https://github.com/yourcove/cove) scraper-pack extensions —
covering scrapers the official [yourcove/communityscrapers](https://github.com/yourcove/communityscrapers)
generator skips (e.g. because they use `subScraper`).

## Scrapers

| Extension id | Site | Scrapes | Port notes |
|---|---|---|---|
| `cove.community.scrapers.yaml.boobpedia` | boobpedia.com | performer by name + by URL | upstream `Image` used a `subScraper` (unsupported in Cove); replaced with the direct infobox image selector (thumbnail resolution) |

## Layout

Mirrors the official repo so packs can be diffed / upstreamed easily:

```
extensions/yaml/<id>/extension.json      # scraper-pack manifest
extensions/yaml/<id>/scrapers/<id>.yml   # ported Stash YAML with attribution header
```

## Installing into Cove

Scraper packs are content-only (no DLL), but **copying the folder is not enough** —
Cove tracks installed extensions in its Postgres `extension_installations` table
(UI/registry installs create the row). A side-loaded pack needs both steps:

```bash
# 1. copy the pack into the extensions dir, folder named by extension id
docker cp extensions/yaml/boobpedia cove:/config/extensions/cove.community.scrapers.yaml.boobpedia

# 2. register it (manifest_json = the pack's extension.json content)
docker exec cove psql -U cove -d cove -c "INSERT INTO extension_installations
  (extension_id, version, enabled, manifest_json, source, categories)
  VALUES ('cove.community.scrapers.yaml.boobpedia', '1.0.0', true,
          '<contents of extension.json>', 'local', 'scraper,metadata,yaml-scraper')
  ON CONFLICT (extension_id) DO UPDATE SET manifest_json=EXCLUDED.manifest_json;"

# 3. restart
docker restart cove
```

If Cove sits behind a Varnish cache, restart Varnish after Cove — Varnish resolves
the backend IP at startup and a recreated/restarted Cove container may get a new one.

## Porting more scrapers

1. Take the upstream `.yml` from stashapp/CommunityScrapers.
2. Check for blockers the official generator enforces: `action: script`/`stash`,
   `subScraper`, or postProcess steps outside `replace`, `parseDate`, `map`,
   `feetToCm`, `lbToKg`. Rework those parts (or accept a degraded field, as with
   Boobpedia's image).
3. Add the AGPL attribution header, write the manifest (copy an existing one and
   update id/name/description/scraperFiles/permissions.network).
4. `permissions.network` must list every host the scraper touches (bare host,
   `www.` variant, and `*.domain.tld`).

## License

AGPL-3.0, inherited from stashapp/CommunityScrapers. See upstream for original
scraper authorship.
