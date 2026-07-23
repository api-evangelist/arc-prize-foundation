---
name: arc-agi-3-play-game
description: Open a scorecard, start an ARC-AGI-3 game, drive it with actions, and close the scorecard to record a score.
api: ARC-AGI-3 REST API
generated: '2026-07-18'
method: generated
source: openapi/arc-prize-foundation-arc-agi-3-openapi.yaml
operations:
  - listGames
  - openScorecard
  - resetGame
  - action1
  - action2
  - action3
  - action4
  - action5
  - action6
  - action7
  - getScorecard
  - closeScorecard
---

# Play an ARC-AGI-3 Game

Drive one full ARC-AGI-3 play session against the REST API at
`https://three.arcprize.org`. Every request sends the `X-API-Key` header, and
game-session requests must echo the `AWSALB*` cookies the server returns (session
affinity). Stay under 600 requests/minute.

## Steps

1. **Discover a game** — call `listGames` (`GET /api/games`) and pick a `game_id`.
2. **Open a scorecard** — call `openScorecard` (`POST /api/scorecard/open`),
   optionally attaching `source_url`, `tags`, and `opaque` metadata. Keep the
   returned `card_id`.
3. **Start the session** — call `resetGame` (`POST /api/cmd/RESET`) with `game_id`
   and `card_id`. Keep the returned `guid` and the `frame`/`available_actions`
   from the `FrameResponse`.
4. **Act in a loop** — inspect `frame` and `available_actions`, then issue the
   chosen command: `action1`–`action5` or `action7` (simple, no coordinates) or
   `action6` (`POST /api/cmd/ACTION6`) with `x`,`y` in the 0–63 range. Include a
   `reasoning` string under 16 KB. Each response is a new `FrameResponse`; repeat
   until `state` is terminal or a level is won.
5. **Reset between levels** — call `resetGame` again with the same `guid` to
   continue within the scorecard.
6. **Read progress** — call `getScorecard` (`GET /api/scorecard/{card_id}`) any
   time to poll aggregate stats.
7. **Close the scorecard** — call `closeScorecard` (`POST /api/scorecard/close`)
   with `card_id` to lock and return the final results.

## Error handling

- **400** — check `game_id`, that `guid` belongs to the active session, ACTION6
  coordinates are within 0–63, and `reasoning` is under 16 KB.
- **401** — supply a valid `X-API-Key` (register at https://arcprize.org/platform).
- **404** — the `card_id` (or its `game_id` entry) does not exist.
- **429** (`RATE_LIMIT_EXCEEDED`) — back off below 600 RPM.
