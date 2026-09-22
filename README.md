# GunSpec MCP Server

The hosted [Model Context Protocol](https://modelcontextprotocol.io) server for the [GunSpec firearms API](https://gunspec.io). It gives Claude, Cursor, VS Code, Codex and other MCP clients 40 read-only tools for firearm specifications, manufacturers, calibers, ammunition and attachment compatibility, including what fits a given firearm, computed from its mounting interfaces.

Nothing to install and nothing to run. Point your client at the server and sign in.

> The server is hosted and closed source. This repository is its public listing: how to connect, what each tool does, and where to report a problem. It is regenerated from the GunSpec monorepo on every SDK release, from the same tool manifest the server serves.

## Connect

### Sign in (recommended)

```
https://mcp.gunspec.io/mcp-oauth
```

Your client sends you to a GunSpec consent screen. Approve it and the client holds a token; no key is copied anywhere. The connection is listed under Profile > API keys > Connected agents, where disconnecting it ends access.

In Claude Code:

```bash
claude mcp add --transport http gunspec https://mcp.gunspec.io/mcp-oauth
```

### Or send an API key

For a client that cannot sign in, or an agent running on a server, send a key in the `X-API-Key` header to the keyed endpoint. Create a key at [app.gunspec.io/keys](https://app.gunspec.io/keys).

```bash
claude mcp add --transport http gunspec https://mcp.gunspec.io \
  --header "X-API-Key: $GUNSPEC_API_KEY"
```

For a client configured with an `mcpServers` file:

```json
{
  "mcpServers": {
    "gunspec": {
      "url": "https://mcp.gunspec.io",
      "headers": {
        "X-API-Key": "${env:GUNSPEC_API_KEY}"
      }
    }
  }
}
```

Step-by-step setup for each supported client is in the [install guide](https://docs.gunspec.io/en/mcp/install).

## Limits

Every tool call is an API request made with your key. It counts against your plan's daily requests, and against a lower daily allowance for MCP calls. Listing the tools is free.

| Plan | Requests per day | MCP calls per day | Tools callable |
| --- | ---: | ---: | ---: |
| Explorer | 50 | 20 | 24 of 40 |
| Builder | 2,000 | 500 | 34 of 40 |
| Studio | 10,000 | 2,500 | 40 of 40 |
| Enterprise | 50,000 | 10,000 | 40 of 40 |

A tool your plan does not cover answers with an error naming the plan it needs. Details: [limits](https://docs.gunspec.io/en/mcp/limits), [errors](https://docs.gunspec.io/en/mcp/errors).

## Tools

Each tool makes one API request. Full arguments and a verified example call for each are on the [tools page](https://docs.gunspec.io/en/mcp/tools).

| Tool | What it answers | API operation | Plan |
| --- | --- | --- | --- |
| `gunspec_changelog` | Lists recent changes to the API and catalog, newest first. | `GET /v1/changelog` | Any plan |
| `gunspec_api_operation` | Returns what the GunSpec API reference documents for an endpoint: its parameters and accepted values, the plan it needs, whether a key is required, caching, every error status with the error.reason values to branch on, and the languages a sample is printed in. | `GET /v1/docs/operations` | Explorer |
| `gunspec_attachment_offers` | Lists sellers currently offering an attachment, with price, stock and an outbound link. | `GET /v1/attachments/{id}/offers` | Explorer |
| `gunspec_caliber_firearms` | Returns every firearm chambered for a cartridge, with pagination. | `GET /v1/calibers/{id}/firearms` | Explorer |
| `gunspec_code_sample` | Returns the code sample the GunSpec API reference prints for an endpoint in one language, exactly as printed, with when it was last checked against production. | `GET /v1/docs/samples` | Explorer |
| `gunspec_firearm_media` | Returns every image, silhouette, render, schematic and 3D model URL for a firearm, with credits and sizes. | `GET /v1/firearms/{id}/media` | Explorer |
| `gunspec_firearm_offers` | Lists sellers currently offering a firearm, with the price in integer minor units plus currency, a stock flag and an outbound link. | `GET /v1/firearms/{id}/offers` | Explorer |
| `gunspec_firearm_variants` | Returns the variants and derivatives of a firearm, for example the Gen 3, Gen 4, Gen 5, MOS and compact forms of a Glock 19. | `GET /v1/firearms/{id}/variants` | Explorer |
| `gunspec_get_attachment` | Returns the full record for one attachment: its requirements (any-of within a group, all-of across groups), what it provides once fitted, caliber ratings, weight, length and sources. | `GET /v1/attachments/{id}` | Explorer |
| `gunspec_get_caliber` | Returns the full record for one cartridge: SAAMI/CIP dimensions, case shape, projectile, origin, parent cartridge and provenance. | `GET /v1/calibers/{id}` | Explorer |
| `gunspec_get_manufacturer` | Returns one manufacturer, including founding details, country, headquarters, description and website. | `GET /v1/manufacturers/{id}` | Explorer |
| `gunspec_list_attachments` | Lists suppressors, optics, magazines, grips, stocks and other attachments. | `GET /v1/attachments` | Explorer |
| `gunspec_list_calibers` | Lists cartridges in the catalog with bullet diameter, case length and type. | `GET /v1/calibers` | Explorer |
| `gunspec_list_categories` | Lists the firearm categories, such as pistol, revolver, rifle, shotgun and submachine gun, with their slugs and counts. | `GET /v1/categories` | Explorer |
| `gunspec_list_firearms` | Lists firearms using structured filters: manufacturer, cartridge, category, action, country, status and year range. | `GET /v1/firearms` | Explorer |
| `gunspec_list_interfaces` | Lists the mount interface standards the fit engine uses: threads, rails, magazine wells, stock interfaces and optic footprints. | `GET /v1/interfaces` | Explorer |
| `gunspec_list_manufacturers` | Lists manufacturers in the catalog with their slugs, optionally filtered by country. | `GET /v1/manufacturers` | Explorer |
| `gunspec_manufacturer_firearms` | Returns every firearm a manufacturer makes or has made, with pagination. | `GET /v1/manufacturers/{id}/firearms` | Explorer |
| `gunspec_plan_limits` | Returns what each GunSpec plan allows, from the configuration the API enforces: requests per minute, per day and per month, MCP calls per day, paging depth, whether lists include a total count, how long to wait after a 429, and the headers a key is sent in. | `GET /v1/docs/limits` | Explorer |
| `gunspec_read_docs` | Returns one GunSpec documentation guide as Markdown. | `GET /v1/docs/guides/{id}` | Explorer |
| `gunspec_search_docs` | Searches the GunSpec documentation guides and returns the sections that answer a question, best first, each with a snippet and a link. | `GET /v1/docs/guides/search` | Explorer |
| `gunspec_similar_firearms` | Returns firearms similar to this one in role, cartridge, size and era. | `GET /v1/firearms/{id}/similar` | Explorer |
| `gunspec_stats_summary` | Returns the current number of firearms, manufacturers, calibers and categories in the catalog. | `GET /v1/stats/summary` | Explorer |
| `gunspec_ammunition_ballistics` | Returns velocity, energy, drop and time of flight at a set of distances for one ammunition load, adjusted for barrel length. | `GET /v1/ammunition/{id}/ballistics` | Builder |
| `gunspec_compare_firearms` | Returns side-by-side specifications for two to five firearms, with the differences calculated. | `GET /v1/firearms/compare` | Builder |
| `gunspec_get_firearm` | Returns the full specification of one firearm: dimensions, weight, calibers, action, capacity, years, designer, description, provenance and record version. | `GET /v1/firearms/{id}` | Builder |
| `gunspec_list_ammunition` | Lists factory loads with bullet weight, bullet type, ballistic coefficient and reference velocity. | `GET /v1/ammunition` | Builder |
| `gunspec_resolve_firearm` | Resolves one free-text name ("G19 gen 5", "M4A1") to a single firearm id, with a confidence score and alternatives. | `GET /v1/firearms/resolve` | Builder |
| `gunspec_search_firearms` | Runs a full-text search over the firearm catalog by name, manufacturer, model number or alias. | `GET /v1/firearms/search` | Builder |
| `gunspec_top_firearms` | Ranks the catalog by one measurable statistic: lightest, heaviest, longest range, highest rate of fire, most compact, highest capacity or most powerful. | `GET /v1/firearms/top` | Builder |
| `gunspec_attachment_firearms` | Returns every firearm that one attachment fits, with the supporting evidence for each. | `GET /v1/attachments/{id}/firearms` | Studio |
| `gunspec_firearm_attachments` | Returns the attachments that fit one firearm, grouped by category. | `GET /v1/firearms/{id}/attachments` | Studio |
| `gunspec_firearm_interfaces` | Returns the mount standards a firearm exposes at each position (muzzle thread, rail, magazine well, stock), with the source and confidence of each row. | `GET /v1/firearms/{id}/interfaces` | Studio |
| `gunspec_get_platform` | Returns one platform's interface rows and its member firearms. | `GET /v1/platforms/{id}` | Studio |
| `gunspec_list_platforms` | Lists firearm families (AK-100, AR-15, Glock) with their member counts. | `GET /v1/platforms` | Studio |

## Workflows

A workflow does the work of several tools in one call and returns one answer. Each request it makes counts against your plan. See the [workflows page](https://docs.gunspec.io/en/mcp/workflows).

| Workflow | What it answers | Requests, at most | Plan |
| --- | --- | ---: | --- |
| `gunspec_cartridge_profile` | Returns a complete profile of one cartridge in a single call: its dimensions and pressure, its parent cartridge and the cartridges derived from it, its common loads, and the number of catalog firearms chambered for it. | 5 | Explorer; Builder for the full answer |
| `gunspec_compare_by_name` | Compares two to five firearms, named as the user wrote them ("G19", "P320 Compact"), in a single call. | 6 | Builder |
| `gunspec_identify_firearm` | Converts an informally written firearm name ("G19 gen 5", "M4A1") into one specification card in a single call. | 4 | Builder |
| `gunspec_load_compare` | Compares the ammunition loads for one cartridge fired from the same barrel, in a single call. | 9 | Builder |
| `gunspec_what_fits` | Returns the optics, muzzle devices, stocks, grips and magazines that attach to a firearm, computed from its mounting interfaces, in a single call. | 2 | Studio |

The same list, with every argument schema, is published as [tools.json](https://docs.gunspec.io/mcp/tools.json) and needs no key.

## Evidence

Every tool's example call is sent to the hosted server each night, and the results are public on the [verification page](https://docs.gunspec.io/en/mcp/verification). The server's own checks, covering authentication, plan gates, limits and transport security, are on the [health page](https://docs.gunspec.io/en/mcp/health).

## Links

- Website: [gunspec.io](https://gunspec.io)
- Documentation: [docs.gunspec.io](https://docs.gunspec.io)
- TypeScript SDK, which exports these tools for your own agent: [gunspec-js](https://github.com/gunspec/gunspec-js)
- Python SDK: [gunspec-python](https://github.com/gunspec/gunspec-python)
- Code examples: [gunspec-examples](https://github.com/gunspec/gunspec-examples)

## Support

Open an issue in this repository, or email [support@gunspec.io](mailto:support@gunspec.io).
