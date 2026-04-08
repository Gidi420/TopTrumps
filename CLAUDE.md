# Top Trumps — Friend Edition
## Claude Code Project Prompt

This is a single-file web app (`index.html`) deployed on GitHub Pages with Supabase as the backend. Friends click a shared link, enter their name, upload a photo, answer 16 questions, and receive a personalised Top Trumps card. All cards are saved to Supabase and viewable by the owner via a password-protected admin view.

---

## Stack

- **Frontend**: Single `index.html` — no build step, no framework, vanilla JS + CSS
- **Hosting**: GitHub Pages (repo: `https://github.com/Gidi420/top-trumps`)
- **Database**: Supabase — table called `players`
- **Storage**: Supabase Storage — bucket called `photos`
- **Fonts**: Bebas Neue (display/headings) + DM Sans (body) via Google Fonts
- **Supabase JS**: loaded from `https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2`

---

## Config block

At the top of the `<script>` tag in `index.html`, there are three constants the user must fill in:

```js
const SUPABASE_URL    = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON   = 'YOUR_SUPABASE_ANON_KEY';
const ADMIN_PASSWORD  = 'YOUR_ADMIN_PASSWORD';
const BUCKET_NAME     = 'photos';
const TABLE_NAME      = 'players';
```

Never hardcode real credentials. Always keep these as clearly labelled placeholders unless the user explicitly provides values.

---

## Screens (in order)

1. **Welcome** (`#s-welcome`) — logo, tagline, "Make my card" CTA + "Admin view" link
2. **Name** (`#s-name`) — single text input for first name, enter key submits
3. **Photo** (`#s-photo`) — drag/drop or tap-to-upload image, preview shown, "Skip" option available
4. **Quiz** (`#s-quiz`) — 16 questions, one at a time, progress bar, back/next navigation
5. **Result** (`#s-result`) — generated card displayed, "Copy link" button, "Do another" button
6. **Admin** (`#s-admin`) — password-gated grid of all submitted cards, refresh button

Screen switching is handled by `go(screenId)` which toggles `.active` class and scrolls to top.

---

## Design system

### Colour palette (CSS variables)
```css
--gold: #e8c94a        /* primary accent — all CTAs, card accents, stat numbers */
--gold-dk: #7a5e00     /* dark gold — used on gold backgrounds */
--navy: #0d0d1a        /* page background */
--navy2: #16162a       /* card / input background */
--navy3: #1e1e35       /* hover state */
--navy4: #262640       /* toast background */
--text: #eeeef8        /* primary text */
--muted: #7777aa       /* secondary text, labels */
```

### Typography
- Display/headings: `'Bebas Neue', sans-serif` — used for logo, step titles, card name, stat numbers
- Body: `'DM Sans', sans-serif` — used for everything else
- Never use Inter, Roboto, or system fonts

### Buttons
- `.btn.btn-gold` — gold background, navy text — primary action
- `.btn.btn-outline` — transparent background, white border — secondary action
- Both have `.btn-full` modifier for full width
- Disabled state: `opacity: 0.3`

---

## The card design

The card (`div.trump-card`, width `300px`) has three sections:

### 1. Hero (top, 210px tall)
Full-bleed photo with the player's name overlaid at the bottom.

**With photo:**
- `<img class="tc-hero-img">` fills the hero absolutely, `object-fit: cover`, `object-position: center top`
- Dark gradient overlay (`.tc-hero-grad`) fades transparent→dark from top to bottom so the name is always readable
- Name rendered as large white Bebas Neue text with text-shadow, positioned at bottom-left

**Without photo (initials fallback):**
- Dark navy background (`#0a0a18`)
- Diagonal repeating-linear-gradient lines pattern (subtle gold texture)
- Large faded initials (160px, opacity 0.07) rotated -8deg as background texture
- Medium initials (72px, opacity 0.22) centred as the focal point
- Same gradient overlay and name treatment as the photo version

**Both versions share:**
- `.tc-series` badge (top-left): "Friend Edition · Top Trumps", semi-transparent pill
- `.tc-hero-slash2` — gold diagonal slash cutting across the bottom-left of the hero into the stats area (uses `clip-path: polygon`)
- `.tc-hero-slash` — navy diagonal slash covering the right portion of the bottom (creates the two-tone diagonal divider effect)

### 2. Stats (middle)
8 rows, one per category. Each row:
- Label (9px, uppercase, muted, fixed 88px width)
- Progress track (4px tall, dark navy background)
- Coloured fill bar (each category has its own colour)
- Numeric value (Bebas Neue 16px, gold)

### 3. Footer (bottom)
Thin top border, "All stats / 100" left, "Top Trumps" right. Both in near-invisible dark text.

---

## Categories and colours

```js
const CATS = [
  "Social Power", "Street Smarts", "Chaos Energy", "Brain",
  "Loyalty", "Rizz", "Drama Index", "Survival Instinct"
];

const CAT_COLORS = {
  "Social Power":     "#5b8fe8",
  "Street Smarts":    "#5cb87a",
  "Chaos Energy":     "#e07b3a",
  "Brain":            "#9b7de8",
  "Loyalty":          "#e8b84a",
  "Rizz":             "#e06b9a",
  "Drama Index":      "#e05858",
  "Survival Instinct":"#3abcaa"
};
```

---

## Scoring system

**Critical design principle**: The scoring is deliberately hidden from users. The category names are never shown during the quiz. Answer options are written to be ambiguous — multiple answers feel equally valid or flattering in different ways. Each answer feeds 2–3 categories simultaneously with different point weights.

### Data structure
Each question has:
```js
{
  text: "Question text...",
  opts: [
    { text: "Option A text", scores: { "Category1": 3, "Category2": 1, "Category3": 4 } },
    { text: "Option B text", scores: { "Brain": 5, "Street Smarts": 4 } },
    // ... 5 options total per question
  ]
}
```

### Score calculation (`calcScores()`)
1. For each answered question, look up the chosen option's `scores` object
2. Add each category's points to a running total, increment that category's count
3. Final score = `Math.round((total / count) * 20)` — maps the 1–5 scale to 0–100

This means categories that appear in more questions get more data points and smoother scores. It's intentional.

---

## The 16 questions

These questions are carefully written so that:
- No option is obviously "the best" across all categories
- Choosing the "smart-sounding" option sometimes scores differently than expected
- Some high-Drama Index answers feel like confident/funny self-awareness rather than a negative
- Some low-scoring options on one axis score high on another (e.g. being risk-averse scores high on Survival Instinct)

```js
const QUESTIONS = [
  {
    text: "You're running 20 minutes late to meet a group of people, most of whom you don't know well. What's going through your head?",
    opts: [
      { text: "Genuine dread — I hate walking into a room that's already started", scores: {"Social Power":1,"Rizz":1,"Chaos Energy":3} },
      { text: "Mild stress but I'll text and apologise and it'll be fine", scores: {"Loyalty":3,"Chaos Energy":2,"Social Power":2} },
      { text: "Annoyance at myself — I had a plan and I didn't stick to it", scores: {"Street Smarts":3,"Chaos Energy":1,"Brain":3} },
      { text: "Honestly not much — people understand, and I'll work the room when I get there", scores: {"Social Power":4,"Rizz":4,"Chaos Energy":4} },
      { text: "I'm already composing the story I'll tell when I walk in", scores: {"Social Power":5,"Rizz":5,"Drama Index":4} },
    ]
  },
  {
    text: "A close friend tells you something in confidence. A week later, another friend brings it up casually — clearly someone told them. What do you do?",
    opts: [
      { text: "Say nothing and hope it blows over", scores: {"Loyalty":1,"Drama Index":3,"Brain":1} },
      { text: "Ask the second friend where they heard it — gather information first", scores: {"Brain":4,"Street Smarts":4,"Loyalty":3} },
      { text: "Go straight to the first friend and ask them directly if they said anything", scores: {"Loyalty":4,"Drama Index":4,"Social Power":3} },
      { text: "I'd already noticed something was off and had a hunch before it came up", scores: {"Street Smarts":5,"Brain":5,"Loyalty":4} },
      { text: "I'm very careful about who I tell things to, so this probably means I told the second friend first", scores: {"Drama Index":5,"Chaos Energy":4,"Loyalty":2} },
    ]
  },
  {
    text: "You're in a group of five trying to find somewhere to eat. Twenty minutes in, no decision has been made. What happens?",
    opts: [
      { text: "I wait it out — eventually someone will decide", scores: {"Social Power":1,"Street Smarts":1,"Chaos Energy":2} },
      { text: "I suggest two specific options to narrow it down", scores: {"Brain":3,"Social Power":3,"Street Smarts":3} },
      { text: "I just start walking towards somewhere I know is good", scores: {"Social Power":5,"Street Smarts":4,"Survival Instinct":3} },
      { text: "I've already Googled three options and have a ranked shortlist ready", scores: {"Brain":5,"Street Smarts":5,"Social Power":3} },
      { text: "I'm happy either way — I just don't want to be the reason we're still arguing", scores: {"Loyalty":3,"Drama Index":1,"Chaos Energy":1} },
    ]
  },
  {
    text: "Someone you've been getting on really well with suddenly goes cold — shorter replies, cancels plans. What do you do?",
    opts: [
      { text: "Nothing. If they want space, I'll give it to them — maybe forever", scores: {"Loyalty":2,"Rizz":1,"Drama Index":1} },
      { text: "Overthink it privately and do nothing", scores: {"Brain":2,"Drama Index":3,"Chaos Energy":2} },
      { text: "Send one casual message to check in, then leave the ball in their court", scores: {"Rizz":4,"Loyalty":4,"Social Power":3} },
      { text: "Ask directly what's going on — I'd rather know", scores: {"Loyalty":5,"Brain":4,"Drama Index":4} },
      { text: "I've already worked out why, and I'm weighing up whether to address it or let it dissolve", scores: {"Street Smarts":5,"Brain":5,"Rizz":3} },
    ]
  },
  {
    text: "It's 11pm, you have work tomorrow, and someone you like texts asking if you want to come out. What actually happens?",
    opts: [
      { text: "I say no — I know myself and I'll regret it in the morning", scores: {"Survival Instinct":4,"Brain":3,"Chaos Energy":1} },
      { text: "I say yes immediately without thinking about it", scores: {"Chaos Energy":5,"Rizz":3,"Survival Instinct":1} },
      { text: "I go for an hour, have a great time, stay until 2am", scores: {"Chaos Energy":4,"Social Power":4,"Survival Instinct":2} },
      { text: "I negotiate — suggest a better night or a shorter hangout now", scores: {"Brain":4,"Rizz":4,"Street Smarts":4} },
      { text: "I'm already dressed before I've finished reading the message", scores: {"Rizz":5,"Chaos Energy":5,"Social Power":5} },
    ]
  },
  {
    text: "You're staying somewhere new and the wifi goes down. You need something done online urgently. What happens next?",
    opts: [
      { text: "I wait for it to come back and hope for the best", scores: {"Street Smarts":1,"Survival Instinct":1,"Brain":1} },
      { text: "I hotspot off my phone without really thinking about it", scores: {"Street Smarts":3,"Brain":2,"Survival Instinct":3} },
      { text: "I've already found a workaround before most people have noticed there's a problem", scores: {"Street Smarts":5,"Brain":4,"Survival Instinct":4} },
      { text: "I find a nearby cafe with good wifi and turn it into an excuse to get out", scores: {"Street Smarts":4,"Social Power":3,"Chaos Energy":3} },
      { text: "I ask someone else to sort it for me", scores: {"Social Power":3,"Street Smarts":1,"Rizz":2} },
    ]
  },
  {
    text: "A night out takes a turn — plans collapse, the group splits, something goes wrong. Looking back, what's your honest role in the story?",
    opts: [
      { text: "I was the one trying to hold it together", scores: {"Loyalty":5,"Social Power":4,"Survival Instinct":3} },
      { text: "I was caught in the middle of two situations I didn't start", scores: {"Drama Index":4,"Loyalty":3,"Chaos Energy":3} },
      { text: "I probably suggested the thing that kicked it all off", scores: {"Chaos Energy":5,"Drama Index":4,"Social Power":3} },
      { text: "I'd already spotted it unravelling and made a quiet exit at the right moment", scores: {"Street Smarts":5,"Brain":4,"Survival Instinct":5} },
      { text: "I genuinely have no idea — I was just vibing and then suddenly things were chaotic", scores: {"Chaos Energy":4,"Drama Index":5,"Rizz":3} },
    ]
  },
  {
    text: "You're asked to give a short speech or toast with about an hour's notice. How does it go?",
    opts: [
      { text: "Badly — I freeze, go blank, and cut it short", scores: {"Social Power":1,"Rizz":1,"Brain":2} },
      { text: "Fine but forgettable — I keep it brief and safe", scores: {"Social Power":2,"Brain":2,"Rizz":2} },
      { text: "Pretty well — I'm nervous but I have a few good lines", scores: {"Social Power":3,"Rizz":3,"Brain":3} },
      { text: "Good — I read the room, get a laugh, keep it moving", scores: {"Social Power":4,"Rizz":4,"Brain":4} },
      { text: "I'm genuinely one of the better people you could ask for this", scores: {"Social Power":5,"Rizz":5,"Brain":4} },
    ]
  },
  {
    text: "You find out a friend has been telling people a version of a shared story that makes you look worse than you were. What do you do?",
    opts: [
      { text: "Nothing — it's not worth the drama", scores: {"Drama Index":1,"Loyalty":2,"Chaos Energy":1} },
      { text: "Bring it up with them privately — I want to understand why", scores: {"Loyalty":4,"Brain":4,"Drama Index":3} },
      { text: "Start quietly correcting the record with the people who heard it", scores: {"Street Smarts":4,"Drama Index":4,"Social Power":3} },
      { text: "Address it directly and calmly — I'm not going to let a narrative sit unchallenged", scores: {"Loyalty":5,"Brain":5,"Social Power":4} },
      { text: "I'm not surprised — I've been watching this dynamic build for a while", scores: {"Street Smarts":5,"Brain":5,"Drama Index":5} },
    ]
  },
  {
    text: "You're travelling alone in a country where you don't speak the language and something goes wrong — flight cancelled, bag missing, wrong city. What's your first move?",
    opts: [
      { text: "Panic first, then eventually find someone to help me", scores: {"Survival Instinct":1,"Street Smarts":1,"Brain":1} },
      { text: "Find the nearest airport/station staff and explain the situation", scores: {"Street Smarts":3,"Social Power":2,"Survival Instinct":3} },
      { text: "Google everything I can in the next five minutes, then act", scores: {"Brain":4,"Street Smarts":4,"Survival Instinct":3} },
      { text: "I've already thought through contingencies — I know what to do", scores: {"Brain":5,"Street Smarts":5,"Survival Instinct":4} },
      { text: "Turn it into an adventure — something will come together and it'll make a great story", scores: {"Chaos Energy":5,"Social Power":4,"Survival Instinct":3} },
    ]
  },
  {
    text: "Someone in your group is being subtly unkind to a quieter member — nothing explosive, just a low-level edge. Most people haven't noticed. What do you do?",
    opts: [
      { text: "Notice it but stay out of it — it's not my place", scores: {"Loyalty":1,"Social Power":2,"Drama Index":2} },
      { text: "Check in privately with the person on the receiving end", scores: {"Loyalty":5,"Social Power":3,"Drama Index":2} },
      { text: "Say something light in the moment that deflects without escalating", scores: {"Social Power":5,"Brain":4,"Rizz":4} },
      { text: "Bring it up with the person responsible when we're one-on-one", scores: {"Loyalty":4,"Brain":5,"Drama Index":3} },
      { text: "I'd have already said something the moment it started — I don't let things like that slide", scores: {"Loyalty":5,"Social Power":4,"Drama Index":4} },
    ]
  },
  {
    text: "You've been seeing someone casually for a few weeks. Things are good but undefined. How do you handle it?",
    opts: [
      { text: "Quietly panic about what it means and say nothing", scores: {"Rizz":1,"Brain":2,"Drama Index":3} },
      { text: "Drop subtle hints and see if they pick them up", scores: {"Rizz":3,"Brain":2,"Chaos Energy":3} },
      { text: "Enjoy it and let it develop naturally — not everything needs a label", scores: {"Rizz":4,"Brain":3,"Chaos Energy":2} },
      { text: "Bring it up lightly and gauge their reaction — I'd rather know where I stand", scores: {"Rizz":4,"Brain":4,"Loyalty":4} },
      { text: "I'm usually the one setting the tone of these things, not waiting to see what they become", scores: {"Rizz":5,"Social Power":5,"Brain":4} },
    ]
  },
  {
    text: "Your group is planning a trip. Three people want completely different things. How does it get resolved?",
    opts: [
      { text: "I stay out of the debate — I'm happy with whatever", scores: {"Social Power":1,"Chaos Energy":2,"Loyalty":3} },
      { text: "I try to find a compromise option that no one loves but everyone accepts", scores: {"Brain":3,"Loyalty":4,"Social Power":3} },
      { text: "I pitch a specific idea that I genuinely believe covers most of what people want", scores: {"Brain":4,"Social Power":4,"Street Smarts":4} },
      { text: "I've already found and sent a place that somehow works for everyone before the argument gets going", scores: {"Brain":5,"Street Smarts":5,"Social Power":5} },
      { text: "I quietly back one person's idea and shift the group dynamic in that direction", scores: {"Social Power":5,"Street Smarts":4,"Brain":4} },
    ]
  },
  {
    text: "You've had a genuinely bad week. Who finds out, and how?",
    opts: [
      { text: "Nobody — I don't share that kind of thing", scores: {"Loyalty":2,"Drama Index":1,"Social Power":2} },
      { text: "One or two people who I specifically chose to tell", scores: {"Loyalty":4,"Brain":3,"Drama Index":2} },
      { text: "Whoever asked — I don't broadcast it but I'm honest if someone checks in", scores: {"Loyalty":3,"Drama Index":2,"Social Power":3} },
      { text: "It comes out in a story I'm telling — I find it easier to make it funny", scores: {"Social Power":4,"Rizz":4,"Drama Index":3} },
      { text: "More people than I intended — I process out loud and don't always clock who's listening", scores: {"Chaos Energy":4,"Drama Index":5,"Social Power":3} },
    ]
  },
  {
    text: "You're trying to impress someone — a date, an employer, someone you admire. What's your approach?",
    opts: [
      { text: "Be as prepared as possible and hope it comes across well", scores: {"Brain":3,"Rizz":2,"Street Smarts":2} },
      { text: "Just be myself and see if it clicks — I can't really fake it", scores: {"Rizz":3,"Loyalty":3,"Social Power":2} },
      { text: "Find out what matters to them beforehand and lean into that", scores: {"Street Smarts":5,"Brain":4,"Rizz":3} },
      { text: "Relax and let them come to me — I find trying too hard always backfires", scores: {"Rizz":5,"Social Power":4,"Brain":3} },
      { text: "I try to understand them well enough to be genuinely interesting to that specific person", scores: {"Brain":5,"Rizz":5,"Street Smarts":4} },
    ]
  },
  {
    text: "Three years from now, which of these is most likely to be true?",
    opts: [
      { text: "Something in my life will be completely different from what I'd have predicted today", scores: {"Chaos Energy":5,"Survival Instinct":2,"Brain":2} },
      { text: "I'll have followed through on at least one thing most people thought was a bad idea", scores: {"Chaos Energy":4,"Social Power":3,"Brain":4} },
      { text: "The people around me will have expanded — I'll know people I haven't met yet who matter a lot", scores: {"Social Power":5,"Rizz":4,"Loyalty":3} },
      { text: "I'll have quietly figured something out that other people are still confused about", scores: {"Brain":5,"Street Smarts":5,"Survival Instinct":4} },
      { text: "Someone in my life will owe me one — and they'll know it", scores: {"Loyalty":5,"Street Smarts":4,"Drama Index":4} },
    ]
  },
];
```

---

## Key rules when modifying this project

1. **Never add a category label above/near the question** — it defeats the hidden-scoring design
2. **Never reorder options so that higher-scoring ones come last** — this is gameable
3. **Questions must feel like genuine personality/behaviour prompts** — not trivia or skill tests
4. **The card design must keep the photo as the hero** — it's the most important visual element
5. **The gold diagonal slash divider between photo and stats must be preserved** — it's the signature design detail
6. **All stats display as integers 0–100** — never show decimals
7. **The admin view is password-protected** — never make it publicly accessible via URL
8. **Photo upload is optional** — the initials fallback must always look intentional, not broken

---

## Supabase database schema

```sql
create table players (
  id uuid default gen_random_uuid() primary key,
  name text not null,
  photo_url text,
  scores jsonb,
  created_at timestamptz default now()
);

alter table players enable row level security;
create policy "Anyone can insert" on players for insert with check (true);
create policy "Anyone can read" on players for select using (true);
```

Storage bucket: `photos`, public, with INSERT and SELECT policies set to `true`.

---

## GitHub workflow

- Repo: `https://github.com/Gidi420/top-trumps`
- Branch: `main`
- Deploy: GitHub Pages from `main` branch root
- Live URL: `https://gidi420.github.io/top-trumps`
- Only file that needs to be in the repo: `index.html` (and this `CLAUDE.md`)
- To deploy changes: edit `index.html`, commit, push — GitHub Pages auto-updates within ~1 minute

---

## Common tasks and how to do them

### Add a new question
Add an object to the `QUESTIONS` array following the exact format above. Assign scores across 2–3 categories. Make sure no single answer is obviously the "best" across all categories.

### Change a category name
Update it in: `CATS` array, `CAT_COLORS` object, and every question's `scores` object where it appears.

### Change the card width
Update `--cw` in `:root`. Currently `300px`.

### Change the hero photo height
Update the `height` on `.tc-hero`. Currently `210px`. Also update `.tc-hero-placeholder` if it has its own height.

### Add a new screen
1. Add HTML with `<div id="s-newscreen" class="screen">` 
2. Navigate to it with `go('s-newscreen')`
3. Add any back/forward buttons that call `go()` with the appropriate screen id

### Change the admin password
Update `ADMIN_PASSWORD` in the config block at the top of the script.
