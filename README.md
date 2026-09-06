# Part Verifier — setup guide

A single-file web app. No server code, no paid APIs, no account required.
It uses a free open-source vision model (CLIP) that runs **inside the phone's
browser** to compare a live camera photo against reference photos you provide,
and reports the closest matching part number + a confidence score.

## What it can and can't do

**Can:** catch the wrong part, a part in the wrong orientation, or something
that visually doesn't resemble any stored reference.

**Can't:** measure exact dimensions or tolerances from a phone photo. That
needs calipers, a CMM, or a depth-camera 3D scan compared to the CAD file —
no free camera-only method can reliably do that.

## 1. Put it online (required — phone cameras need HTTPS)

Browsers block camera access on plain HTTP pages (except `localhost`), so you
need to host this file somewhere with HTTPS. All of these are free:

**Easiest — Netlify Drop**
1. Go to https://app.netlify.com/drop
2. Drag the `index.html` file onto the page.
3. You get a live HTTPS link instantly (e.g. `random-name.netlify.app`).
4. Open that link on your phone.

**Alternative — GitHub Pages**
1. Create a new GitHub repo, upload `index.html` (rename to keep it as `index.html`).
2. Repo Settings → Pages → set source to the main branch.
3. GitHub gives you a URL like `https://yourname.github.io/reponame/`.

**Alternative — Vercel**
1. https://vercel.com → New Project → drag-and-drop the file/folder.

Once hosted, open the link on your phone, allow camera access when prompted,
and (ideally) add it to your home screen so it opens like an app.

## 2. Add reference photos

1. Open the **References** tab.
2. For each part number, capture or upload 3–6 photos covering the angles
   someone is likely to check it from (iso, front, side, top). Label each one.
3. If you don't have the physical part yet, open the STEP/STL file in
   **FreeCAD** or **Blender** (both free), rotate to the correct view, and
   export a screenshot/render as a PNG. Upload that the same way — it works
   as a reference image just like a camera photo.
4. Optionally add a "wrong orientation" example for parts that are commonly
   installed backwards — the app will flag a match against that as a reject.

The first time the page loads, it downloads the vision model (~90 MB, one
time only, then cached by the browser). After that it works fully offline
for scanning and adding references (only hosting the page itself needs
internet, once loaded).

## 3. Scan a part

1. Open the **Scan** tab, point the camera at the part, fill the frame, avoid
   glare, and tap **Capture & Check**.
2. It shows the best-matching part number, a confidence percentage, and the
   next few closest references so you can sanity-check the call.
3. Confidence roughly means:
   - 85%+ → confident match (or confident match to a known reject example)
   - 65–85% → possible match, worth a manual look
   - below 65% → no confident match found

These thresholds are a starting point — after real use, if you're getting
false positives/negatives, they can be tuned in the `showResult` function
in `index.html` (search for `0.85` and `0.65`).

## 4. Back up / share your reference library

- **Export library** (References tab) downloads a single `.json` file with
  every reference photo and its computed fingerprint. Save this file to
  OneDrive as your backup / source of truth.
- **Import library** on any device loads that file back in — no retraining,
  no recomputation needed. This is how you'd roll the same reference set out
  to multiple phones/stations.

## Notes for scaling past 50 parts

The matching approach (CLIP embeddings + nearest neighbor) doesn't require
retraining as you add parts — just add more reference photos and they're
immediately searchable. If you eventually have many hundreds of parts and
matching starts to feel slow or less precise, the next upgrade would be a
proper vector database (e.g. free tier of Supabase or Qdrant) instead of
comparing against every stored embedding in the browser — happy to help with
that step if/when you get there.
