# Zalo Command Relay Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot that receives Zalo admin commands and forwards messages to specified Zalo groups/personal pages. Owner manages whitelist, views logs, and retries failures via Telegram commands.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram owner
- Zalo group/page recipients

## Success criteria

- Immediate message delivery to Zalo targets
- Delivery logs with status tracking
- Owner can retry failed deliveries

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu
- **/status** (command, actor: user, command: /status) — View system status
- **/recent** (command, actor: user, command: /recent) — View recent deliveries
- **/retry** (command, actor: user, command: /retry) — Retry failed delivery by ID

## Flows

### Command Handling
_Trigger:_ Zalo admin message

1. Validate admin ID
2. Parse target and content
3. Forward to Zalo
4. Log delivery

_Data touched:_ Admin ID list, Delivery record

### Owner Retry
_Trigger:_ /retry <id>

1. Validate owner identity
2. Locate delivery record
3. Resend message
4. Update status

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Admin ID list** _(retention: persistent)_ — Whitelist of Zalo IDs allowed to send commands
  - fields: id, added_at
- **Delivery record** _(retention: persistent)_ — Message delivery metadata
  - fields: id, sender_id, target, timestamp, status
- **Error log** _(retention: session)_ — Failed delivery records
  - fields: delivery_id, error_type, timestamp

## Integrations

- **Zalo** (required) — Message delivery to groups/personal pages
- **Telegram** (required) — Owner control and notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Manage admin ID whitelist
- Retry failed deliveries
- View delivery logs

## Notifications

- Delivery confirmation to owner
- Error alerts for failed messages

## Permissions & privacy

- Zalo API access limited to whitelisted admins
- Telegram owner ID is strictly private

## Edge cases

- Invalid admin ID in command
- Failed message resend after error
- Missing target ID in command

## Required tests

- End-to-end command flow with Zalo delivery
- Retry command with status update

## Assumptions

- Admin whitelist is fixed and owner-managed
- Message delivery is immediate
- CTA buttons limited to 5 per message
