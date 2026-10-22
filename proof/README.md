# What was actually run, and what it did

Six real n8n executions, saved whole. Every file here is a complete execution: every node's input and output,
exactly as it ran on 2026-10-21 (executions 246–251). Not screenshots, and not a description of a test.

The runs are production-mode executions through a live webhook. That matters for one reason: n8n only persists
`$getWorkflowStaticData` outside manual runs, so a manual run could not prove the bot remembers the previous
turn. Run 4 is the one that needs it.

**The setup.** n8n 2.35.7, self-hosted. The model is `qwen3:4b-instruct` through Ollama, running locally, free,
and Apache-2.0, so free for commercial use too. These runs replace the 2026-09-27 set, made on `qwen2.5:3b`, whose
licence does not allow commercial use without the publisher's permission. The demo
shop is "Hide & Seam", four lines of notes:

    - We sell handmade leather bags. Orders ship in 2-3 working days.
    - We ship to the UK and the EU. We do not ship outside the EU.
    - Standard delivery is 3-5 days, tracked. Express is next working day.
    - Returns: 30 days, unused, buyer pays return postage.

The two Telegram nodes are disabled in the test workflow, because this machine has no bot token. n8n passes a
disabled node's input through, so everything above and below them still ran and still got logged. The workflow
under test is derived from the shipped file by `build/maketest.py` rather than written separately, so the two
cannot drift apart.

## The six runs

| run | the customer says | what it did | right? |
|---|---|---|---|
| 1 | how long does delivery take? | answered: "Standard delivery takes 3-5 days, tracked. Express is next working day." | yes |
| 2 | do you ship to Australia? | answered: "We do not ship outside the EU, so we do not ship to Australia." | yes |
| 3 | do you have these in green? | said it could not answer. The notes say nothing about colours | yes |
| 4 | and is that the tracked one? | answered "Yes, standard delivery is tracked." — and it only knew what "that" meant because it remembered run 1 | yes |
| 5 | can I speak to someone please | said it could not answer | yes |
| 6 | this is the third time I have asked. useless. | said it could not answer | yes |

Six of six.

## Run 2, the one the old model got wrong

The notes say "We do not ship outside the EU." So "do you ship to Australia?" has an answer, and the answer is
no. On the old default, `qwen2.5:3b`, this run said it could not answer, even though the reason it wrote down
("the customer asks about a place the notes exclude from shipping, which is outside the EU") showed it had worked
the answer out. It set the wrong flag. `qwen3:4b-instruct` answers it.

That miss was not fixed in code, and it would have been easy to: the reason text was right there and could be
pattern-matched into flipping the answer. That would mean guessing on the customer's behalf from a string the
model wrote, which is the one thing this workflow is built not to do. It still fails the same safe way when the
model is unsure: the customer gets the sentence you wrote, and the bot invents nothing.

Six right out of six is six cases, not a promise. A small model running free on your own machine will get some
of your customers' questions wrong; that is why every run is here to read.

## What these runs do not prove

- **Nothing here went over Telegram.** The two Telegram nodes are disabled, so what is proved is every decision
  and every message this workflow composes, not Telegram's delivery of it. The paid version has live Telegram
  runs in its own proof folder.
- **One model, one shop.** Six cases against four lines of notes on `qwen3:4b-instruct`. Your notes and your model will
  behave slightly differently, which is why the runs are here to read rather than summarised in a number.
- **No handoff, because there is not one.** This is the free version. It answers or it does not, and when it does
  not, nobody is told. Getting the conversation sent to you, the bot going quiet while you deal with it, and
  handing it back with /resume are the paid version.

## Running it again yourself

    python3 build/build.py        # writes telegram-lite.json from source
    python3 build/run_cases.py    # posts all six cases at n8n and rewrites this folder

Needs n8n on 127.0.0.1:5678 and Ollama with the model named in Settings.
