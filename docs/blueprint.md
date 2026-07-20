# StreakMate — Bot specification

**Archetype:** workflow

**Voice:** encouraging and warm — write every user-facing message, button label, error, and empty state in this voice.

Private Telegram bot for habit tracking with gentle reminders, one-tap check-ins, and weekly progress summaries. Focuses on streak maintenance without punitive language, with full data privacy and customizable schedules.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- habit builders
- privacy-conscious individuals

## Success criteria

- Users maintain and extend streaks through consistent check-ins
- Weekly summaries show measurable progress over time
- No duplicate habit records due to one-tap confirmation logic

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open onboarding flow or main menu
  - inputs: Telegram session data
  - outputs: Greeting message with privacy explanation
- **Create New Habit** (button, actor: user, callback: habit:new) — Start habit creation wizard
  - inputs: habit title, scheduling rule, preferred time
  - outputs: Habit confirmation message
- **/habits** (command, actor: user, command: /habits) — Show all active habits with status
  - inputs: user preferences
  - outputs: Habit list with streak metrics
- **Mark Done** (button, actor: user, callback: occurrence:done) — Confirm completion of scheduled habit
  - inputs: habit ID, current date
  - outputs: Streak update confirmation

## Flows

### Onboarding
_Trigger:_ /start

1. Greet user with privacy statement
2. Collect timezone and locale
3. Create first habit with scheduling rules

_Data touched:_ User, Habit

### Daily Reminder
_Trigger:_ scheduled local time

1. Send reminder message with action buttons
2. Handle one-tap check-in
3. Prevent duplicate entries

_Data touched:_ Occurrence, SeriesMetrics

### Weekly Summary
_Trigger:_ configured weekly time

1. Calculate weekly completion metrics
2. Generate encouraging summary message
3. Include milestone celebrations

_Data touched:_ SeriesMetrics, User

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user profile and preferences
  - fields: telegram_id, display_name, locale, timezone, notification_window, quiet_hours, privacy_flags
- **Habit** _(retention: persistent)_ — User-defined habit with scheduling rules
  - fields: id, owner, title, description, scheduling_rule, preferred_time, status, created_at, paused_since
- **Occurrence** _(retention: persistent)_ — Daily habit completion record
  - fields: habit_id, date, status, timestamp, source, prevented_duplicate_flag
- **SeriesMetrics** _(retention: persistent)_ — Streak and completion statistics
  - fields: current_streak, best_streak, completion_rate, milestones

## Integrations

- **Telegram** (required) — Direct message notifications and inline buttons
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Create/edit/delete habits
- Pause/resume habits
- Configure notification preferences
- Export data as CSV
- Delete account

## Notifications

- Daily reminders at user-configured local time
- Weekly summary with progress visualization
- Milestone celebration messages
- Supportive messages for missed days

## Permissions & privacy

- All data stored privately per user
- No third-party sharing
- User controls data deletion/export
- Telegram-only communication channel

## Edge cases

- Timezone changes during active streak
- Duplicate check-in attempts
- Paused habit resumption after multiple days
- Milestone notifications when offline

## Required tests

- End-to-end onboarding flow with habit creation
- Daily reminder with duplicate prevention
- Weekly summary generation with milestone detection
- Privacy mode verification

## Assumptions

- Timezone inferred from Telegram metadata if not set
- Default scheduling rule is daily
- Week defined as Monday-Sunday for consistency
