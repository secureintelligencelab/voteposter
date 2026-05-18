# 🏆 PosterVote — Campus Edition

A **zero-build** poster session voting system that runs directly from GitHub Pages.
No Node.js, no build step, no server — just two HTML files.

---

## 🆕 What Changed (Campus Edition)

| Old (Class-based) | New (Campus-based) |
|---|---|
| `classes/{classId}` (Firestore) | `campuses/{campusId}` (Firestore) |
| "Class" in UI | "Campus" in UI |
| Guest QR / Guest Code | Teacher QR / Teacher Code |
| `?class=ID` in URL | `?campus=ID` in URL |
| One overall result | **Best Poster per Campus** winner banner |
| CSV: `name, id, email` | CSV: `name, id, email, campus` |

**Backwards compatible**: vote.html still reads `?class=` links and the legacy `classes` Firestore collection automatically.

---

## 🚀 Deploy in 5 Minutes

### Step 1 — Fork / Upload to GitHub

1. Create a new GitHub repository (e.g. `poster-vote`)
2. Upload both files: `index.html` and `vote.html`
3. Go to **Settings → Pages → Branch: main → / (root)** → Save
4. Site live at: `https://YOUR_USERNAME.github.io/poster-vote/`

### Step 2 — Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com) → **Add project**
2. Enable **Firestore Database** (test mode for setup)
3. Enable **Authentication → Sign-in method → Email/Password**
4. Create admin user: **Authentication → Users → Add user**
5. **Project Settings → Your Apps → Add web app** → copy config

### Step 3 — Paste Firebase Config

In both `index.html` and `vote.html`, find the `firebaseConfig` block near the top of the `<script>` and paste your values.

---

## 🏫 Campus Workflow

```
Admin (index.html)                      Students / Teachers (vote.html?campus=ID)
──────────────────                      ────────────────────────────────────────
1. Create one Campus per location
2. Import CSV (with optional campus col) ──► Only matching campus rows imported
3. Create groups, assign students       ──► Student cannot vote own group
4. Set Teacher Code (for judges)
5. Open voting, share links     ─────────►  Student: enters PIN
                                            Teacher: enters Teacher Code
                                            Both: rate Design / Novelty / Content ★
6. View Results ◄────────────────────────── Votes saved per campus
   ► Best Poster winner banner shown
     for each campus independently
7. Close voting
```

---

## 📋 CSV Format

```csv
name,id,email,campus
Alice Johnson,20210001,alice@university.edu,Main Campus
Bob Smith,20210002,bob@university.edu,City Campus
Carol Lee,20210003,carol@university.edu,Main Campus
```

**campus column is optional.** If omitted, all rows are imported into the current campus.
If present, only rows matching the campus name (case-insensitive) are imported — others are skipped with a count shown.

You can also upload a **global student list** to each campus panel — the campus filter does the splitting automatically.

---

## 🔐 Firestore Security Rules

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /campuses/{campusId} {
      allow read: if true;
      allow write: if request.auth != null;

      match /students/{studentId} {
        allow read: if true;
        allow write: if request.auth != null;
        allow update: if request.auth == null
          && request.resource.data.diff(resource.data).affectedKeys()
             .hasOnly(['hasVoted','votedForGroupId']);
      }

      match /groups/{groupId} {
        allow read: if true;
        allow write: if request.auth != null;
      }

      match /votes/{voteId} {
        allow read: if request.auth != null;
        allow create: if true;
        allow update, delete: if request.auth != null;
      }
    }

    // Legacy class-based rules (keep if upgrading from old version)
    match /classes/{classId} {
      allow read: if true;
      allow write: if request.auth != null;
      match /students/{s}{ allow read: if true; allow write: if request.auth != null; }
      match /groups/{g}{ allow read: if true; allow write: if request.auth != null; }
      match /votes/{v}{ allow read: if request.auth != null; allow create: if true; allow update,delete: if request.auth != null; }
    }
  }
}
```

---

## 📁 Files

| File | Purpose |
|------|---------|
| `index.html` | Admin panel — campuses, students, groups, voting control, results |
| `vote.html` | Voting page — student PIN auth or teacher code auth + scoring form |

---

## 🗺 Application Flow Summary

- **One campus = one voting session = one Best Poster winner**
- Students log in with a 6-char PIN; they **cannot vote for their own group**
- Teachers log in with the Teacher Code; they can vote for all groups
- Results page shows a **🏆 Best Poster** winner banner per campus, plus ranked leaderboard
- Auto-refresh every 15 seconds in live mode
