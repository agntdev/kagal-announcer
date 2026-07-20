# Kagal Announcer Bot — Bot specification

**Archetype:** community

**Voice:** professional and warm — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that manages subscriptions to a community channel, delivering read-only announcements to subscribers via direct messages while handling opt-ins, opt-outs, and delivery retries.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Members of the kagal community seeking non-interactive announcement subscriptions

## Success criteria

- Subscribers receive announcements within 5 minutes of source channel posting
- Subscription status updates reflected in <2 seconds
- 99% delivery success rate after retries

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Display welcome message with subscription status and Subscribe/Unsubscribe buttons
- **/stop** (command, actor: user, command: /stop) — Trigger unsubscription flow
- **/status** (command, actor: user, command: /status) — Show current subscription status and subscriber count
- **/help** (command, actor: user, command: /help) — Display command list and basic usage

## Flows

### Onboarding
_Trigger:_ /start

1. Display welcome message
2. Show subscription status
3. Offer Subscribe/Unsubscribe buttons

_Data touched:_ Subscriber

### Announcement Delivery
_Trigger:_ New message in source channel

1. Format message content
2. Send to all subscribers
3. Log delivery records
4. Retry failed deliveries up to 3 times

_Data touched:_ Announcement, Delivery record

### Unsubscription
_Trigger:_ /stop or Unsubscribe button

1. Confirm unsubscription
2. Remove subscriber from database

_Data touched:_ Subscriber

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Subscriber** _(retention: persistent)_ — Telegram user subscription records
  - fields: telegram_user_id, display_name, subscription_timestamp, opt_in_source
- **Announcement** _(retention: persistent)_ — Source channel message metadata
  - fields: source_message_id, timestamp, text, media_metadata
- **Delivery Record** _(retention: persistent)_ — Announcement delivery tracking
  - fields: announcement_id, subscriber_id, delivery_timestamp, status

## Integrations

- **Telegram** (required) — Bot API messaging and channel monitoring
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure admin chat ID for alerts
- Adjust data retention periods (announcements: 90 days default, subscribers: 365 days default)

## Notifications

- Admin receives error alerts and subscription count updates
- Delivery failure retries tracked in delivery records

## Permissions & privacy

- Subscriber data stored securely with 90-day announcement retention
- Admin chat configured by owner for operational alerts

## Edge cases

- Telegram user ID changes requiring subscription update
- Source channel message containing unsupported media formats
- Subscriber blocking bot mid-delivery cycle

## Required tests

- End-to-end subscription lifecycle test
- Announcement delivery with media fallback
- Retry logic for failed deliveries

## Assumptions

- Owner will grant bot access to source channel messages
- Default retention periods meet privacy requirements
- Admin will monitor configured alert channel
