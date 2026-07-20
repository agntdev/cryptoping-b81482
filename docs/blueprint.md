# CryptoPing: Personal Crypto Price Tracker — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

CryptoPing is a private Telegram bot that lets users track cryptocurrency prices with customizable alerts. Users can set price thresholds or percent change triggers for specific time windows, receive instant notifications (with quiet hours suppression), check prices manually via /price, and enable optional morning summaries. The bot owner gains access to usage metrics showing active users and most frequently triggered alerts.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Private crypto investors/traders
- Bot owner for analytics

## Success criteria

- Users receive accurate price alerts based on thresholds/percent changes
- Owner accesses real-time usage metrics via /admin_stats
- All notifications respect user-configured quiet hours and suppression rules

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open onboarding menu with quick action buttons
- **Add coin** (button, actor: user, callback: add_coin:start) — Initiate coin addition flow with common coin buttons and ticker input
- **View watchlist** (button, actor: user, callback: watchlist:view) — Show current watchlist with management options per entry
- **Set alerts** (button, actor: user, callback: alerts:configure) — Configure threshold or percent-based alerts for selected coins
- **Quiet hours** (button, actor: user, callback: quiet_hours:configure) — Set notification suppression time window
- **Morning summary** (button, actor: user, callback: summary:configure) — Enable/disable daily price digest and set delivery time
- **/price** (command, actor: user, command: /price) — Show full watchlist prices or specific coin price with percent change

## Flows

### onboarding
_Trigger:_ /start

1. Display welcome message with quick action buttons
2. Prompt for initial coin selection or manual ticker entry

_Data touched:_ User

### add_coin
_Trigger:_ add_coin:start

1. Show BTC/ETH/TON buttons + 'Enter ticker' option
2. Validate ticker with fuzzy matching
3. Confirm coin details before adding to watchlist

_Data touched:_ User, Watchlist entry

### manage_alerts
_Trigger:_ alerts:configure

1. Select coin from watchlist
2. Choose alert type (threshold/percent)
3. Set price/percent values and direction
4. Confirm alert rule details

_Data touched:_ Watchlist entry, Threshold alert, Percent alert

### price_check
_Trigger:_ /price

1. Parse optional ticker parameter
2. Fetch current price and percent change
3. Format and return results with watchlist summary if no ticker specified

_Data touched:_ Watchlist entry, Price feed

### admin_stats
_Trigger:_ /admin_stats

1. Verify owner identity
2. Aggregate active users and top triggered alerts
3. Display metrics in formatted table

_Data touched:_ User, Sent alert record

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user profile with preferences and settings
  - fields: telegram_id, display_name, timezone, quiet_hours_start, quiet_hours_end, summary_time, pause_state
- **Watchlist entry** _(retention: persistent)_ — Monitored cryptocurrency with alert rules
  - fields: user_id, ticker, display_name, last_known_price, threshold_alerts, percent_alerts
- **Threshold alert** _(retention: persistent)_ — Price threshold alert rule
  - fields: type, target_price, cooldown_period
- **Percent alert** _(retention: persistent)_ — Percent change alert rule
  - fields: direction, percent_value, time_window, minimum_reversal
- **Sent alert record** _(retention: persistent)_ — Record of alerts sent to suppress repeats and track metrics
  - fields: user_id, watch_entry_id, alert_type, timestamp

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- /admin_stats (owner-only command)

## Notifications

- Price threshold alerts with old/new price values
- Percent change alerts with direction and magnitude
- Morning summary digest at user-configured time
- Quiet hours suppression with queued alerts

## Permissions & privacy

- All user data is private and never shared
- Owner can only view aggregate metrics, not individual watchlists
- User must explicitly confirm ambiguous ticker additions

## Edge cases

- Unknown tickers with fuzzy suggestions
- Price API failures with retry logic
- Overlapping alert rules for same coin
- Quiet hours during morning summary window

## Required tests

- Add coin flow with typo handling
- Alert suppression during quiet hours
- /price command with valid/invalid tickers
- Admin metrics visibility restrictions

## Assumptions

- 1-hour default window for percent alerts
- BTC/ETH/TON as pre-configured buttons
- 12-hour default cooldown for threshold alerts
- Owner uses Telegram ID for admin access
