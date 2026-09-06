# Billinge Running Club — Weekly Route Page

A single web page that automatically shows **this week's** running route on a map, with
distance/elevation stats and a **Download GPX** button — so members *and* visitors can grab
the route for their watch or phone.

- **You never edit the page.** Each week, someone just uploads the route file to a shared
  Google Drive folder.
- **Files are named by week number:** `36.gpx`, `37.gpx`, … (ISO week, 1–52).
- **Optionally add a run name** after the number: `37-Electric Dog.gpx` makes the page show
  the route titled **"Electric Dog"**. (A plain `37.gpx` works too — it just has no title.)
- The page reads the correct file straight from Google Drive and always shows the
  **upcoming Wednesday run** (see below).

Live page (once set up): **https://gpx.billingerunning.club**

---

## Weekly routine (for the whole team)

1. Open the shared **WeeklyRoutes** Google Drive folder.
2. Upload the route named by its **week number**, optionally with a run name:
   - `37.gpx` — just the route, no title.
   - `37-Electric Dog.gpx` — the page shows the run titled **"Electric Dog"**.
   - If a file for that week already exists (from last year), just replace/overwrite it.
   - Not sure of the week number? It's shown at the top of the live page.
3. Done. The page updates itself.

> Naming rules: start with the week number, then an optional `-` or space and the run name.
> `37.gpx`, `37-Electric Dog.gpx`, `37 Electric Dog.gpx` and zero-padded `07-Foo.gpx` all
> work. Keep it to one route file per week.

### Which week shows by default?

The page is built around the **Wednesday run**:

- On **run day (Wednesday)** it shows **that day's** route.
- From **Thursday onwards** it automatically rolls forward to **next Wednesday's** run, so
  people arriving after a run see the *upcoming* one to prep for.
- The **week number is the ISO week of the run date** (e.g. the run on Wed 2 Sep 2026 is
  week 36; Wed 9 Sep is week 37).
- Visitors can still use **← Prev / Next →** to browse other weeks, and **Upcoming run** to
  jump back to the default.

Runs on a different day? Change `runDay` in the `CONFIG` block of `index.html`
(`0`=Sunday … `3`=Wednesday … `6`=Saturday).

---

## One-time setup (do this once)

You need three things: a **Drive folder**, a **Google API key**, and **GitHub Pages**
hosting on the `gpx.billingerunning.club` address.

### 1. Google Drive folder
1. Create a folder (a **Shared Drive** works well for a club) and add the people who upload
   routes as **editors**.
2. Set link sharing to **Anyone with the link → Viewer**.
   - On a Google Workspace account an admin may need to allow external ("anyone with link")
     sharing for this folder.
3. Open the folder and copy its **Folder ID** from the URL:
   `https://drive.google.com/drive/folders/`**`THIS_LONG_ID`**

### 2. Google API key (read-only)
1. Go to <https://console.cloud.google.com/> and create a **new project** (e.g. "BRC GPX").
2. **APIs & Services → Library →** search **Google Drive API →** **Enable**.
3. **APIs & Services → Credentials → Create credentials → API key.** Copy the key.
4. Click the key to restrict it (recommended):
   - **Application restrictions → Websites (HTTP referrers)** → add
     `https://gpx.billingerunning.club/*`
     *(while testing you can also add your GitHub Pages URL, e.g.
     `https://YOURNAME.github.io/*`).*
   - **API restrictions → Restrict key →** tick **Google Drive API** only.
   - Save.

> The key only ever reads **public** files, so it's safe to ship in the page. The
> restrictions stop it being reused on other sites.

### 3. Configure the page
Open `index.html`, find the `CONFIG` block near the bottom, and paste your values:

```js
const CONFIG = {
  FOLDER_ID: "PASTE_FOLDER_ID",   // from step 1.3
  API_KEY:   "PASTE_API_KEY",     // from step 2.3
  routeColor: "#209D50"
};
```

### 4. Host it on GitHub Pages (free)
1. Create a free account at <https://github.com> if you don't have one.
2. Create a new **public** repository (e.g. `brc-gpx-site`).
3. Upload `index.html` and the `CNAME` file (drag-and-drop into the repo → **Commit**).
4. **Settings → Pages →** Source: **Deploy from a branch**, Branch: **main** / **/(root)**
   → **Save**.
5. In the same page, under **Custom domain**, enter `gpx.billingerunning.club` and save,
   then tick **Enforce HTTPS** (may take a few minutes to become available).

### 5. Point the domain at GitHub Pages (DNS)
Wherever the DNS for `billingerunning.club` is managed, add a record:

| Type  | Host / Name | Value                    |
|-------|-------------|--------------------------|
| CNAME | `gpx`       | `YOURNAME.github.io`     |

(Use your GitHub username. DNS can take up to an hour to take effect.)

Once live, share **https://gpx.billingerunning.club** on WhatsApp each week — the link
never changes.

---

## Optional: show it inside the Squarespace site

Because the page is styled to match the club site, you can embed it on a Squarespace page
so it lives under the main navigation. On a page, add a **Code** block (needs a Business
plan or higher):

```html
<iframe src="https://gpx.billingerunning.club"
        style="width:100%; height:900px; border:0;" loading="lazy"></iframe>
```

Adjust the height to taste. This is optional — the standalone link works on its own.

---

## Testing without the live setup

Open `sample-route.gpx` to see the expected file format. To preview the page locally with a
real route before wiring up Drive, you can temporarily paste a GPX string into the browser
console:

```js
renderGpx(await (await fetch('sample-route.gpx')).text(), { title: "Electric Dog" }, DEFAULT_RUN);
```

*(Local `fetch` of a file needs the page served over http — e.g. VS Code's Live Server — or
just deploy to GitHub Pages and test there.)*

---

## How it works (for whoever maintains this)

- `index.html` is fully self-contained (HTML + CSS + JS) and uses two free libraries from a
  CDN: **Leaflet** (map) and **leaflet-gpx** (GPX parsing + distance/elevation).
- It works out the upcoming Wednesday run, lists the folder via the Google Drive API,
  matches the file whose name starts with that week number, downloads it, draws it on an
  OpenStreetMap map, reads any title from the filename, and offers it for download.
- **Prev / Next / Upcoming run** buttons let viewers browse other weeks; if a week's file
  isn't uploaded yet, a friendly "no route uploaded yet" message shows.
- No server, no database, no page edits — just the weekly file upload to Drive.
