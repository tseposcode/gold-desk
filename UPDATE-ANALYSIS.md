# How Claude updates the Gold Desk analysis

The app (`index.html`) shows whatever is in `analysis.json`. A scheduled Claude task refreshes that file on weekdays. Each run:

1. Note the current time in UTC and SAST (UTC+2).
2. Search the web for the latest XAUUSD (spot gold) news and prices from today and the last 1-2 days. Cover what moves gold: the US dollar (DXY), US yields, Fed expectations, major data and events, geopolitics, central-bank and ETF flows, and price action. Prefer dated reporting from the last 24 hours.
3. Rewrite `analysis.json` in exactly the current shape:
   - `updated`: now, ISO 8601 UTC. This is what the app shows as "Last refresh".
   - `asof`: the time of the latest price you found, ISO 8601 UTC.
   - `price`: the latest spot price as a number.
   - `bias`: `direction` is `up`, `down` or `neutral`. `label` is short, e.g. "Leaning up above 4,075".
   - `event`: the next high-impact event with its SAST time, or `""`.
   - `bullets`: 5-7 items of `{title, text}`. The title is 2-3 words and the text is one plain sentence.
   - `levels`: 7-10 items of `{price, type, note}`, ordered from the highest price to the lowest. `type` is `resistance`, `now` or `support`, with exactly one `now` row for the current price.
   - `buy` and `sell`: each has `title`, numeric `entry`, `stop` and `target`, and `keys` (4 items of `{label, value}`: zone, stop, targets, cancelled if). `look` has 4-5 items on what confirms the trade. `watch` has 4 items on what can go wrong, including news and discipline mistakes like moving the stop. Put entries, stops and targets at real levels, with reward at least 2x the risk where the levels allow it. The plan against the main bias must say it is counter-trend.
   - `sources`: the pages you used, as `{name, url}`.
4. Rules: use only what your searches show. Mark any number you could not confirm with "UNVERIFIED". Never invent levels. Give no win probabilities and no guarantees, because this is decision support, not a signal service. Use plain, short sentences and SAST times.
5. Check that the file is valid JSON (`python3 -m json.tool analysis.json`), then commit it to `main` with the message `Update gold analysis <SAST date time>` and push.
6. On Saturday and Sunday the gold market is closed. Do nothing.
