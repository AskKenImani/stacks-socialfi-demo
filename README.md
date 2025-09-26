# SocialFi on Bitcoin — README + Wireframes

> **Project:** Stacks SocialFi — store content-hash on Stacks, Firebase for profiles & media, tipping with STX.

## Quick pitch

A simple, beautiful SocialFi app built on Stacks + Bitcoin. Users connect with their Stacks wallet, create short posts (text + optional image). Full post content is stored off-chain (Firebase Storage) and a content-hash (SHA-256 or IPFS CID) is recorded on-chain via a Clarity contract. Users can tip other users with STX using stacks.js wallet flows.


## Contents of this document

1. What you will build (MVP)
2. Architecture diagram
3. Sprint-by-sprint breakdown with step-by-step tasks (Validate → Build → Pitch)
4. How to set up local dev (frontend, Clarinet, Firebase)
5. File tree & starter code snippets (Clarity contract, React components, firebase.js)
6. Wireframe sketches (text / ASCII) you can copy into Figma
7. Deployment & submission checklist (DoraHacks)
8. Pitch script (2–3 minutes)


## 1) MVP features (must-haves for hackathon)

* Wallet connect (Stacks) and login flow
* Create post: text + optional image upload to Firebase Storage
* Compute SHA-256 of post content (or use uploaded file URL) and call Clarity contract to record `postId -> contentHash` and author
* Feed: show posts by combining Firebase cached data with on-chain proof check
* Tip: let users tip post authors via STX (stx-transfer in Clarity or direct wallet transfer)
* Deployed demo on Firebase Hosting + public GitHub repo + README + pitch video

## 2) Architecture (high level)

```
[User Browser] --(connects via stacks.js/connect)--> [Stacks Wallet (Hiro, Xverse)]
      |                                                   |
      |--(uploads image/text)--> [Firebase Storage]        |--(signs txs & sends)-->
      |                                                   v
      |--(writes metadata)--> [Firebase Firestore]    [Stacks Blockchain]
      |                                                   ^
      |--(call create-post tx with hash)--------------/  
      |                                                   |
      v                                                   |
   [Frontend (React + Vite + stacks.js)] ------------------+
                 |--(reads on-chain via Hiro Stacks API)-->

```

Notes: All on-chain storage is minimal (content hash only). Firebase (Spark plan) is used for profiles, avatars, media, and fast feed queries.

---

## 3) Sprint-by-sprint

### Sprint 1 — Validate (Sept 26 – Oct 2) — Goal: solid plan, wireframes, repo

**Day 1: Project scaffolding**

* Create a GitHub repo named `stacks-socialfi-demo`
* Create a `README.md`
* Create an issues checklist in GitHub with these buckets: `sprint-1-validate`, `sprint-2-build`, `sprint-3-pitch`.

**Day 2: Wireframes & UX (precise steps)**

* Open Figma (free plan) → create a new project called **SocialFi Wireframes**.
* Recreate 5 screens based on ASCII below: Landing, Connect Wallet, Feed, Create Post, Profile.
* Add basic flows: Connect Wallet → Feed; Feed → Create Post; Feed → Profile.
* Annotate each wireframe: where the wallet connect button sits, where the verified badge appears, and where the tip button goes.
* Export PNGs of the wireframes and upload them into your GitHub repo under `/docs/wireframes/`.

**Day 3: Firebase & Accounts**

* Create a Firebase project

  * Enable Firestore (start in test mode while developing) and Storage.
  * In Firestore create these collections: `users`, `posts` (cache), `pendingTxs`.
* Create a Firebase Hosting site

**Day 4: Local dev environment**

* Create the Vite React skeleton:

  ```bash
  npm create vite@latest stacks-socialfi -- --template react
  cd stacks-socialfi
  npm install
  ```
* Install required libs:

  ```bash
  npm install firebase @stacks/connect @stacks/network @stacks/transactions @stacks/blockchain-api-client
  ```

**Day 5: Write the project plan / README (precise steps)**

**Deliverable for Sprint 1:** GitHub repo with README, wireframes (PNG in `/docs/`), Firebase project created.


### Sprint 2 — Build (Oct 3 – Oct 9) — Goal: working demo + deployed site

**Day 1: Clarinet + Clarity contract (local dev)**

* Install Clarinet (follow Clarinet README). Create a `contracts/` folder.
* Add a simple Clarity contract `socialfi.clar`. The contract stores `post-id -> (author, hash)`.
* Use Clarinet to run local devnet and unit tests.

**Day 2: Wallet connect & auth**

* Add a `ConnectWallet` component using `@stacks/connect` (showConnect). Test connecting with Hiro wallet testnet.
* After connecting, show the user's Stacks address and profile (from Firestore `users` collection).

**Day 3: Off-chain storage + hashing**

* Implement file/image upload to Firebase Storage (client-side). After upload, compute SHA-256 of the post text (use `crypto.subtle.digest`) and create a compact hex string or 32-byte buffer.
* Save a cached post entry to Firestore `posts` with fields: `postId`, `author`, `contentUrl` (firebase storage URL), `hashHex`, `createdAt`, `onChain: false`.

**Day 4: Make on-chain `create-post` transaction**

* Use `@stacks/transactions` and `@stacks/connect` to create a contract call transaction to your deployed contract on testnet.
* Pass `postId` and `bufferCV` (from hex) as arguments.
* When transaction is broadcasted, update Firestore `pendingTxs` with the txid and map to `postId`.

**Day 5: Read & confirm on-chain**

* Use Hiro Stacks API to poll transaction status. When confirmed, mark `posts.onChain = true` and display a verified badge.

**Day 6: Tipping feature**

* Add a small UI button `Tip` on posts. On click, open the wallet to send an STX transfer to the author's address (use `stxTransfer`).
* Provide UX for small tips (0.1 STX, 1 STX) and confirm success.

**Day 7: Polish & deploy**

* Run `npm run build` and `firebase init hosting` then `firebase deploy --only hosting` to publish.

**Deliverable for Sprint 2:** Live demo link, GitHub commits, working post flow (upload → hash → on-chain reference) and tipping.


### Sprint 3 — Pitch (Oct 10 – Oct 16) — Goal: submit to DoraHacks with video

**Day 1-2: Make a short demo video**

* Record a 2–3 minute video: 30s problem, 60–90s demo (create post + show on-chain proof + tipping), 30s why built on Bitcoin & next steps.

**Day 3: Final README & docs**

**Day 4-5: Final testing**

* Smoke test the whole flow. Make sure storage rules are ok (test mode) and you show that on-chain data ties to the cached post.

**Day 6: DoraHacks submission**

**Deliverable for Sprint 3:** DoraHacks submission, demo live, 2–3 minute pitch video.


## 4) Local dev setup (concise commands)

**Frontend skeleton**

```
npm create vite@latest stacks-socialfi -- --template react
cd stacks-socialfi
npm install
npm install firebase @stacks/connect @stacks/network @stacks/transactions @stacks/blockchain-api-client
```

**Clarinet (local contract testing)**
Follow Clarinet quickstart (install via cargo or npm where provided):

```
# clone clarinet and use it, or install from releases per Clarinet README
clarinet new
clarinet test
clarinet devnet
```

**Firebase (hosting deploy)**

```
npm run build
firebase init hosting # pick your project
firebase deploy --only hosting
```


## 5) File tree & starter snippets

```
stacks-socialfi/
├─ contracts/
│  └─ socialfi.clar
├─ frontend/
│  ├─ src/
│  │  ├─ main.jsx
│  │  ├─ App.jsx
│  │  ├─ components/
│  │  │  ├─ ConnectWallet.jsx
│  │  │  ├─ CreatePost.jsx
│  │  │  └─ Feed.jsx
│  │  └─ firebase.js
├─ clarinet.toml
├─ package.json
└─ README.md
```

### Minimal Clarity contract (contracts/socialfi.clar)

```lisp
(define-map posts ((post-id uint)) ((author principal) (hash (buff 32))))

(define-public (create-post (post-id uint) (content-hash (buff 32)))
  (begin
    (map-set posts ((post-id post-id)) ((author tx-sender) (hash content-hash)))
    (ok post-id)
  )
)

(define-read-only (get-post (post-id uint))
  (map-get posts ((post-id post-id)))
)
```

Use Clarinet for tests and then deploy to testnet.

### ConnectWallet.jsx (minimal)

```jsx
import React from 'react';
import { showConnect } from '@stacks/connect';

const appDetails = { name: 'Stacks SocialFi', icon: '' };

export default function ConnectWallet() {
  const onConnect = () => {
    showConnect({ appDetails, onFinish: () => window.location.reload() });
  };
  return <button onClick={onConnect}>Connect Stacks Wallet</button>;
}
```

---

## 6) Wireframe sketches

### Landing

```
+----------------------------------------------------+
| LOGO            Stacks SocialFi     [Connect Wallet] |
| -------------------------------------------------- |
| Hero: "Post to Bitcoin-backed SocialFi"           |
| [Create Post]   [View Feed]   [Profile]            |
+----------------------------------------------------+
```

### Feed

```
+----------------------------------------------------+
| [Create Post Box]                                 |
| Post Card:                                         |
|  - Avatar  Username  [Verified badge if onChain]  |
|  - Content (text + small image)                    |
|  - [Tip] [View on-chain tx link]                  |
+----------------------------------------------------+
```

### Create Post

```
+----------------------------------------------------+
| Text input (max 280 chars)                        |
| [Upload Image]  [Post (upload -> pending -> onchain)] |
+----------------------------------------------------+
```

### Profile

```
+----------------------------------------------------+
| Avatar Username                                   |
| Bio                                               |
| User's posts (list)                               |
| Tip balance / received tips                       |
+----------------------------------------------------+
```

## 7) DoraHacks submission checklist

* [ ] GitHub repo public with final README
* [ ] Live demo URL (Firebase Hosting)
* [ ] 2–3 minute pitch video (YouTube unlisted or Vimeo)
* [ ] Short appraisal of what was built & future work
* [ ] Attach README + link in DoraHacks form
