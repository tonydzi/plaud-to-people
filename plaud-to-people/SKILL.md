---
name: plaud-to-people
description: "Turn Plaud recordings (meetings, networking events, calls) into a clean transcript, who-said-what, episode map per person, and people notes with evidence. Triggers: /plaud-to-people, 'разбери запись Plaud', 'кто был на ивенте', 'process my Plaud recording', 'who did I talk to'."
version: 1.0.0
license: MIT
---

# /plaud-to-people: Plaud recording → transcript → speakers → people notes

Works with Claude Code, Codex, Cursor, Gemini CLI or any agent that can run shell commands.
Built and battle-tested on real networking evenings (5-13 hours of audio a day, 2 languages).

## What you need

1. A Plaud device and account.
2. Plaud's official CLI: `npm install -g @plaud-ai/cli`, then `plaud login`.
3. Optional, for your own transcription and diarization (much better than the built-in one, see Step 2):
   `pip install faster-whisper soundfile` and a GPU (CPU works, just slower).
   For speaker diarization: `pip install "nemo_toolkit[asr]"` (Sortformer) or `pyannote.audio`.

## Step 1. Pull the recordings

```bash
plaud today                 # list of today's recordings
plaud file <id>             # metadata
plaud transcript <id> -o rec.txt
plaud summary <id>
plaud audio <id>            # download link, lives ~24h: download right away
```

⚠️ **`start_at` from Plaud is UTC.** Your photos and calendar are local time. Convert with a named
time zone (`America/Los_Angeles`), never a fixed offset: a fixed `-7` is an hour off after DST changes.

⚠️ **One recording ≠ one meeting.** Plaud splits long sessions into files back to back,
and one file often holds several different people.

## Step 2. Treat Plaud's transcript as a draft, not as truth

Measured on our recordings: the built-in transcript text covered only **7-47% of the recording duration** (no human gold transcript, so not a proven speech-recall figure),
sometimes hallucinated (Chinese characters inside a Russian/English talk) and mangled names
("Gwern" → "Warren"). Fine for a quick summary, not for deciding who someone is.

Your own pass:

```python
from faster_whisper import WhisperModel
m = WhisperModel("large-v3", device="cuda", compute_type="float16")  # device="cpu", compute_type="int8" without GPU
segs, info = m.transcribe("rec.ogg", language=None, vad_filter=True,
                          multilingual=True)   # detect language per 30s window
for s in segs:
    print(f"[{s.start/60:05.1f}m] {s.text}")
```

⛔ **Never pin the language** (e.g. `language="ru"`) on a mixed-language event: the model will
silently *translate* English speech and invent names ("Danfeng" became a Russian-sounding name).
Smooth text, wrong facts: worse than a gap.

Quick sanity metrics: CJK characters in a non-Chinese talk ≈ hallucination; the same phrase repeated
many times in a row = the model looping on silence. If the recording was in a pocket and very quiet,
normalize volume (RMS to about -20 dBFS) before transcribing.

## Step 3. Diarization: who is speaking

Run a diarizer (Sortformer, pyannote) on 5-minute windows. Be honest about its limits:
- Sortformer handles **at most 4 speakers** per window. At a crowded event it hits that ceiling all the time.
- Labels from different windows are **not** the same people (`speaker_0` in window 5 ≠ `speaker_0` in window 6).
  To link them across the day, take a voice embedding per speaker per window (e.g. NeMo TitaNet)
  and cluster by cosine similarity.
- Find **yourself** by a few reference phrases you know are yours. Do not assume "the voice present
  in every recording is me": in our test that was the friend who stood next to me all evening.

Use diarization as a **second method on top of meaning**: build boundaries from the text,
check them with speaker labels.

## Step 4. Episode map: one conversation = one episode

For each recording write `episodes/<id>.json`: start, end, channel (`in-person` / `phone` / `self-talk`),
and **evidence with timestamps** (country, company, role, what they build, contact they gave).

Fast way to find boundaries, because at events people say names out loud:

```bash
grep -nE "what'?s your name|my name is|I'?m [A-Z]|nice to meet|where are you from" rec.txt
grep -nE "raising|pre-?seed|it'?s called|whatsapp|linkedin|telegram|scan me" rec.txt
```

⭐ **The rule that matters most: the machine never assigns a name on its own.**
The agent proposes episodes, timestamps and evidence; a **human confirms** who the person is.
Two different "Dennis" in your contacts plus an automatic match = a business conversation glued
to the wrong person, and it looks plausible. A name heard from one source only is marked 🤔.

## Step 5. Glue photos and screenshots (optional)

A photo or a screenshot of someone's profile taken at time *t* most likely belongs to whoever spoke
in `[t - 3 min, t + 1 min]`. That is a **candidate**, not a fact. Confirm with a second independent clue
(they said their name, you said it). Refuse the match if the photo falls into two episodes,
the episode was a phone call, or the time zone is unknown.

## Step 6. People notes / CRM

One note per confirmed person: date, event, where you met, why it matters, what is known
(with recording timestamps `mm:ss` as evidence), what is unknown (`🤔 not confirmed`), and the next step.
Personal or medical things heard on the recording do not go into the note.

## Step 7. Follow-up debt: say it out loud

"Scan my QR / take my WhatsApp" does **not** create a chat. Check whether a thread actually exists.
If not, that is the most expensive leftover of the evening: write within 3 days.

Bonus: grep the recording for instructions **you gave to yourself** ("remind me", "send her the summary",
"research this"). They hide in the middle of hour-long files and get lost otherwise.

## Boundaries

No invented names. Nothing is sent to anyone from this skill. Audio is never deleted.
A people note without a source is not "done".

---
Made by Anton Dziatkovskii and Mycroft (his synthetic AI co-founder) at Palo Alto AI Research Lab.
github.com/tonydzi · MIT
