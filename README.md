# Radora

Radora teaches English through reading. You pick topics you like, learn a set of words, then read an article built from those exact words. It's in English, Spanish and Turkish, and it's not out yet.

The code is private. These are the parts I'd want to read about in someone else's app.

## The first study engine was boring

It taught a batch, quizzed the batch in the same order, then replayed the misses. After two rounds you knew what was coming, and that kills recall. Now there's one queue, and each answer puts the word back at a variable distance ahead. The whole thing is a pure reducer with no imports.

## You can't farm a word

One Postgres function schedules every review. A correct answer only counts if the word is due, so ten right answers in one sitting move it up once. Only the first miss of the day costs a level. The client just applies the answer.

## Writes wait for the network

Three MMKV queues hold failed writes and replay them on launch, on reconnect and when the app returns to the foreground. Every queued write was made safe to replay before the queue existed.

## Free users see premium articles without getting them

Recommendation functions return IDs and counters, never sentences. A free user gets a locked card and the text stays behind Row Level Security.

## The app is called Radora and its bundle id isn't

It began as Language Traveler. Changing the slug, bundle id or scheme would mean a new store listing, new OAuth redirects and orphaned crash reports, so only the display name changed. `app.config.ts` says why.

---

React Native, Expo, TypeScript, Supabase, RevenueCat.

Questions: [anl@icodex.dev](mailto:anl@icodex.dev)
