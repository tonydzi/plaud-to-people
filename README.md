# plaud-to-people

A Claude Code / Codex / Cursor skill: **Plaud recording → honest transcript → who said what → one note per person you met.**

## По-русски, коротко

Скилл для тех, кто ходит с Plaud на встречи и ивенты. Он забирает записи через официальный CLI Plaud,
делает свою расшифровку (встроенная у Plaud ловила у нас только 7-47% речи и путала имена),
разбирает, кто говорил, режет вечер на разговоры с конкретными людьми и делает по каждому заметку
с таймкодами-уликами. Главное правило: машина сама имя человеку не ставит, подтверждаете вы.

**Как поставить (2 минуты):**
1. `npm install -g @plaud-ai/cli` и `plaud login`
2. Скопируйте папку `plaud-to-people` в `~/.claude/skills/` (Codex: `~/.codex/skills/`)
3. Скажите агенту: «разбери мою запись Plaud за сегодня»

Для своей расшифровки и диаризации нужен Python (`faster-whisper`, по желанию NeMo или pyannote); без них
скилл работает на тексте Plaud, просто менее точно.

## In English

1. `npm install -g @plaud-ai/cli` and `plaud login`
2. Copy `plaud-to-people/` into `~/.claude/skills/` (or your agent's skills folder)
3. Ask your agent: "process today's Plaud recordings and tell me who I talked to"

What's inside: the lessons we paid for on real 13-hour recording days (UTC timestamps, never pinning the
language, the 4-speaker ceiling of diarizers, never auto-assigning names, photo-to-speaker matching).

Feedback, issues and PRs welcome.

Made by Anton Dziatkovskii and Mycroft (his synthetic AI co-founder), Palo Alto AI Research Lab · github.com/tonydzi · MIT
