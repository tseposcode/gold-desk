# How Claude updates the US100 Desk analysis (v2)

The app (`index.html`) shows whatever is in `analysis.json`. Since v2 the app covers **US100 (Nasdaq 100)** only; v1 (gold) is kept on the branch `v1-gold`. A scheduled Claude task refreshes `analysis.json` on weekdays at about 07:00, 11:00, 15:00 and 19:00 SAST. Each run:

1. Note the current time in UTC and SAST (UTC+2).
2. Search the web for the latest Nasdaq 100 / US100 news and prices from today and the last 1-2 days. Cover what moves US100: big tech and chip stocks (Nvidia, Apple, Microsoft, Amazon, Meta, Alphabet, Tesla, Broadcom, AMD), their earnings, US yields, Fed expectations, the dollar, US data (CPI, NFP, PPI, retail sales, jobless claims, ISM), the VIX, and price action in Nasdaq 100 futures. Prefer dated reporting from the last 24 hours.
3. Rewrite `analysis.json` in exactly the current shape:
   - `market`: always `"US100"`. The app ignores any file without it.
   - `updated`: now, ISO 8601 UTC. This is what the app shows as "Last refresh".
   - `asof`: the time of the latest price you found, ISO 8601 UTC.
   - `price`: the latest Nasdaq 100 futures or index price as a number.
   - `newsDay`: `true` only if CPI, NFP, PPI or retail sales is released today at the US data time (15:30 SAST in US winter, 14:30 in US summer). Otherwise `false`. This switches the app's range to the last 30 minutes before the open.
   - `bias`: `direction` is `up`, `down` or `neutral`. `label` is short, e.g. "Leaning down below 28,300".
   - `event`: the next high-impact event with its SAST time, or `""`.
   - `bullets`: 5-7 items of `{title, text}`. The title is 2-3 words and the text is one plain sentence naming its source.
   - `levels`: 7-10 items of `{price, type, note}`, ordered from the highest price to the lowest. `type` is `resistance`, `now` or `support`, with exactly one `now` row for the current price.
   - `levelNote`: one sentence saying which instrument the levels come from (futures, cash index or a CFD) and that the user's CFD may be offset from futures.
   - `buy` and `sell`: each has `title`, numeric `entry`, `stop` and `target`, and `keys` (4 items of `{label, value}`: zone, stop, targets, cancelled if). `look` has 4-5 items on what confirms the trade, tied to the 16:30 SAST open sweep-and-reclaim where it fits. `watch` has 4 items on what can go wrong, including news and discipline mistakes like moving the stop. Put entries, stops and targets at real levels, with reward at least 2x the risk where the levels allow it. The plan against the main bias must say it is counter-trend.
   - `sources`: the pages you used, as `{name, url}`.
4. Rules: use only what your searches show. Mark any number you could not confirm with "UNVERIFIED". Never invent levels. Give no win probabilities and no guarantees, because this is decision support, not a signal service. Use plain, short sentences and SAST times.
5. Check that the file is valid JSON (`python3 -m json.tool analysis.json`) and that `market` is `"US100"`, then commit it to `main` with the message `Update US100 analysis <SAST date time>` and push.
6. On Saturday and Sunday the US market is closed. Do nothing.
