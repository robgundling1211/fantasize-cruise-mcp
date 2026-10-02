# Fantasize Cruise Finder: an MCP server and API for AI agents

Find the best-value cruise for someone in one call. Fantasize prices every sailing of 23 cruise lines
(about 30,000 sailings) every night, judges each fare **low, typical or high against its own usual price**,
and returns a link the person opens to book that exact sailing.

Free. No key. Hosted, so there is nothing to install.

| | |
|---|---|
| MCP endpoint (streamable HTTP) | `https://fantasize.net/mcp` |
| REST | `https://fantasize.net/api/v1/<tool>` |
| OpenAPI 3.1 | https://fantasize.net/openapi.json |
| Claude Connectors Directory | https://claude.ai/directory/fantasize-cruise-finder |
| Official MCP registry | `net.fantasize/cruise-finder` |
| About the data | https://fantasize.net/llms.txt |

## Connect it

- **Claude**: one click from the Connectors Directory: [Fantasize Cruise Finder](https://claude.ai/directory/fantasize-cruise-finder) (or Customize → Connectors → Add custom connector → `https://fantasize.net/mcp`)
- **ChatGPT**: Settings → Connectors (developer mode) → add `https://fantasize.net/mcp`
- **Any MCP client**:
  ```json
  { "mcpServers": { "fantasize": { "type": "http", "url": "https://fantasize.net/mcp" } } }
  ```
- **Plain language:** pass the person's words as `q`:
  `https://fantasize.net/api/v1/search_cruises?q=alaska+in+july+from+seattle,+balcony,+under+$3000+for+two`
  It is read by fixed rules; the answer says how it was understood and which words were not.
- **No MCP?** Every tool is a plain GET:
  `https://fantasize.net/api/v1/search_cruises?region=alaska&month=2027-07&from_port=Seattle&cabin=balcony`

## Tools

| tool | what it answers |
|---|---|
| `search_cruises` | Ranked sailings by region, month or dates, departure port, a port to visit, line, ship, tier, nights, cabin and budget. Sort by best value (furthest below its usual price), lowest price, lowest per night, or soonest. |
| `get_cruise` | One sailing in full: day-by-day itinerary, each cabin grade's price (and which are sold out), each against its usual, the fare's price history, the same itinerary on other dates, and the best thing to do at each port, with its verified price. |
| `compare_cruise_lines` | Which line is cheapest for the same trip: median and lowest price per night by line, with each line's cheapest sailing. |
| `last_minute_cruise_deals` | Sailings leaving North American ports in the next ~3 weeks, measured against the book-ahead price. |
| `port_guide` | The best things to do at a cruise port: the verified seller, price and operator where we have one, and a booking page for each. |
| `cruise_options` | The regions, lines, departure ports and months covered, with counts. |

## What the numbers mean

- Prices are US dollars **per person, for the whole cruise, cabin for two, taxes and port fees included**
  (Viking River quotes exclude taxes), the cheapest cabin of that grade as sold through Cruisebound, read on `priced_on`.
- `vs_usual` compares the fare per night with the median for the same sailing on other dates, else the same ship
  and length, else the line and length, and says which. No brochure prices, no "was" prices.
- `book_url` opens that exact sailing on fantasize.net, where the person taps **Book**. Agents should give the
  person the link, not try to book.

## Example

```
GET https://fantasize.net/api/v1/search_cruises?region=alaska&month=2027-07&from_port=Seattle&cabin=balcony&limit=1
```
```json
{
  "results": [{
    "line": "Oceania", "ship": "Oceania Riviera", "depart_date": "2027-07-15", "nights": 7,
    "departs_from": "Seattle, Washington",
    "ports_of_call": ["Victoria, British Columbia", "Seattle, Washington", "Icy Strait, Alaska", "Sitka, Alaska", "Ketchikan, Alaska"],
    "quoted_cabin": "balcony", "price_per_person": 2640, "price_per_person_per_night": 377,
    "vs_usual": { "verdict": "low", "percent_vs_usual": -32, "usual_per_person_per_night": 557,
                  "compared_with": "other dates of this sailing" },
    "priced_on": "2026-09-30",
    "book_url": "https://fantasize.net/cruise/oceania-riviera-alaska-from-seattle-7-nights-c91f4744d3c3?depart=2027-07-15&grade=balcony"
  }]
}
```
(Prices change nightly; this one is from 2026-09-30.)

## Limits

150 calls per 10 minutes per caller. Contact: hello@fantasize.net
