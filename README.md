# Telegram AI support bot for n8n (Lite, free)

An n8n workflow that answers your customers on Telegram from notes you write, and only from those notes. If a
question is not covered, it sends a sentence you choose instead of guessing.

It runs on n8n's free self-hosted edition, and by default on a free local model through Ollama (`qwen3:4b-instruct`,
Apache-2.0), so it costs nothing to run. OpenAI or Anthropic work too, with your own key. Built and tested on n8n 2.35.7.

**Download:** [telegram-lite.zip](https://github.com/TuckerCode/n8n-telegram-bot-lite/releases/latest/download/telegram-lite.zip)
(the workflow, the setup guide and six real test runs). The same files are in this repository.

## Set it up in about ten minutes

1. **Import the workflow.** In n8n: Workflows, then Import from File, and pick `telegram-lite.json`.
2. **Connect your bot.** Message @BotFather on Telegram, send `/newbot`, and paste the token it gives you into
   a new Telegram credential on the **Send the answer** node. Pick the same credential on **Say you will come
   back to them**.
3. **Write your notes.** Open the **Settings** node and put the facts your bot may state in `knowledge`, one per
   line. That list is the only thing it is allowed to say.

Before you switch the workflow on, message your new bot once yourself. A Telegram bot cannot start a
conversation, so until you have spoken to it, it cannot reach you.

[SETUP.md](SETUP.md) has every field, how to use OpenAI or Anthropic instead of Ollama, and what to do if n8n
runs in Docker.

## What it does not do

- **It does not hand a chat to a person.** When it cannot answer, the customer gets your sentence and nobody is
  told.
- Text messages only, and it only knows your notes.
- A small local model will get some questions wrong. [proof/README.md](proof/README.md) has every test run,
  and the one the previous default model got wrong.

## More free templates, by email

New free n8n templates go out to the email list first, with a short note on setting each one up:
https://stats.sender.net/forms/dR630O/view

## The full version

**Telegram AI Bot with Human Handoff** ($15) adds what this one leaves out. When a customer needs a person, it
alerts you on Telegram with the whole conversation and the reason it stopped, goes quiet on that chat so it
never talks over you, and gives the chat back when you send `/resume`. It ships with nine real saved runs,
including one where the model got it wrong.

- Full version: https://whop.com/airankings/telegram-ai-bot-with-human-handoff/?ref=gh-lite
- All templates (Telegram, invoice parsing, email triage, or all three for $39):
  https://airankings-templates.whop.site/?ref=gh-lite
