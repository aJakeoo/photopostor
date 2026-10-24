# PHOTOSTOR — Setup Guide (v2)

A photo-based social deduction game. Everyone plays on their own phone. Imposters don't know the word — they have to bluff with a photo.

---

## What's new in v2

- **Sessions of 3 rounds.** Imposters stay imposters across all rounds. Each round picks a *new random word*.
- **No one picks the word** — not even the host. Fully random from a built-in word bank.
- **Eliminations + spectators.** When you're voted off, you become a spectator who can see who the imposter is and watch the rest.
- **Win conditions:** Innocents win by voting out all imposters. Imposters win by surviving 3 rounds.
- **Ties:** "The vote was tied. No one was voted off." Round still counts toward the imposter's survival.
- **Self-voting allowed.**
- **Test mode** for the host: bypass 3-player minimum (works with 2 devices), no win conditions, host ends the session manually.
- **Rebuilt phase advancement** using Firestore transactions — fixes the vote-freeze bug where the game stalled if the host's snapshot listener throttled.
- **Auto-rejoin.** Refresh the tab, accidentally close-and-reopen, or your phone backgrounded too long — you snap right back into the room you were in, on the correct screen. "Leave room" is the only way to actually exit.

---

## What's in this folder

| File | What it does | Do you edit it? |
|------|--------------|-----------------|
| `index.html` | Page structure (screens) | rarely |
| `style.css` | **All visual design** | to restyle |
| `app.js` | Game logic + Firebase sync | rarely |
| `firebase-config.js` | **Your Firebase keys** | ✅ |
| `manifest.json` | Add-to-Home-Screen settings | optional |
| `icon-192.png`, `icon-512.png` | App icons | optional |
| `photos/` | **Photo library** players can pick from | to add photos |

---

## How to play

1. **Host** taps Create Room, gets a 4-letter code.
2. Everyone else taps Join Room and enters the code + their name.
3. Host picks **1–3 imposters** (optionally enables test mode), then Start.
4. Each phone reveals: **green word** for innocents, **red "IMPOSTER"** for imposters.
5. Everyone picks a photo from the library (or uploads their own) and submits it.
6. Photos appear for all. **Talk it out in person.**
7. Host opens the vote.
8. Round result: either someone's eliminated (and becomes a spectator) or the vote was tied.
9. Host hits "Next Round" — new word, same imposters.
10. Session ends when all imposters are out (**innocents win**) or 3 rounds pass with any imposter still alive (**imposters win**).
11. "New Session" returns everyone to the lobby with re-randomized imposters.

---

## Photo library

The photo library is the main way to submit. The submit screen opens straight onto a grid of photos, and uploading your own is a small link underneath.

- **Every player is dealt their own random photos each round:** 12 from the round's category, or 24 from the whole library on the **All photos** tab. Different players see different options.
- **↻ show me different photos** deals a fresh random hand if nothing fits.
- Each category's pool is built from broad searches on the theme, not just the secret words, so the imposter can't work out the word from what's on offer.
- Once a player submits a library photo, it fades out for everyone else that round. No duplicate picks.
- Library picks are saved as a short file path, not image data, so they barely touch Firestore.

**Adding your own photos:** drop a JPG (around 640px is plenty) into `photos/` and add a line to `photos/library.js`, using a category name from the word bank in `app.js`:

```js
{ file: "my-photo.jpg", category: "Ocean & Water" },
```

The starter set is public-domain/CC0 photos from Openverse. Sources are in `photos/CREDITS.md`.

---

## Locking it down (before public sharing)

Replace test rules in Firestore → Rules:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /rooms/{code} {
      allow read, write: if true;
    }
  }
}
```

---

## Notes

- Images compress in-browser to ~900px JPEG. Stays under Firestore's 1MB doc limit even with multiple submissions.
- Phase advancement is now a **distributed transaction** — any client can trigger it when conditions are met. No single point of failure.
- The map variant is still a planned v3, swapping the photo-submit step for a pin drop.

— built for Jake
