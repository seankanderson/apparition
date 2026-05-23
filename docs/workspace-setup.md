# Workspace Setup — How to Organize Your Folders

This is the most important setup step that people skip. Getting this right prevents a lot of confusion later.

---

## The Golden Rule

> **Apparition is a toolkit. Your app is a separate thing.**
> They should never live inside each other.

Think of it like this: Apparition is the instruction manual and toolbox. Your app is the thing you're building. You don't put the thing you're building *inside* the instruction manual.

---

## The Right Folder Structure

```
📁 dev\                        ← a parent folder you create (call it whatever you like)
  │
  ├── 📁 apparition\            ← the Apparition toolkit (cloned once, rarely changed)
  │     ├── README.md
  │     ├── docs\
  │     ├── prompts\
  │     └── agents\
  │
  └── 📁 my-invoice-app\        ← YOUR app (built here, this is where code lives)
        ├── SPEC.md
        ├── schema.sql
        ├── Program.cs
        ├── Pages\
        ├── Api\
        └── Data\
```

You open VS Code at the **`dev\` parent folder** — so you can see both side-by-side in the file explorer.

---

## Step-by-Step Setup (Windows)

### Step 1 — Create a parent folder

Open File Explorer and create a folder for all your dev work. Good options:
- `C:\dev\`
- `C:\projects\`
- `C:\Users\YourName\projects\`

It doesn't matter what you name it. Pick something short and easy to type.

### Step 2 — Put Apparition inside it

If you haven't cloned Apparition yet, open a terminal and run:
```powershell
cd C:\dev
git clone https://github.com/your-org/apparition.git
```

If you already have an Apparition folder somewhere else, move it into your parent folder using File Explorer drag-and-drop.

### Step 3 — Create your app folder as a sibling

When the AI builds your app, it will create a new folder. That folder should go *next to* the apparition folder — not inside it.

The AI will suggest a command like:
```powershell
cd C:\dev
mkdir my-invoice-app
cd my-invoice-app
```

Or you can create the folder in File Explorer and tell the AI where it is.

### Step 4 — Open VS Code at the parent folder

This is the key step. Open the **parent folder** in VS Code so you can see both folders:

1. Open VS Code
2. Go to **File → Open Folder**
3. Select `C:\dev\` (or whatever you named your parent folder)
4. Click **Select Folder**

You should now see both `apparition` and `my-invoice-app` in VS Code's left sidebar.

---

## Step-by-Step Setup (Mac)

### Step 1 — Create a parent folder

Open Terminal and run:
```bash
mkdir ~/dev
```

Or use Finder to create a folder named `dev` in your home directory.

### Step 2 — Put Apparition inside it

```bash
cd ~/dev
git clone https://github.com/your-org/apparition.git
```

### Step 3 — Your app folder goes next to it

```bash
cd ~/dev
mkdir my-invoice-app
```

### Step 4 — Open VS Code at the parent folder

```bash
code ~/dev
```

Or in VS Code: **File → Open Folder → select your `dev` folder**.

---

## What It Looks Like in VS Code

When set up correctly, your VS Code Explorer panel (left sidebar) shows:

```
EXPLORER
  ▼ DEV
    ▶ apparition
    ▶ my-invoice-app
```

You can click into `apparition` to read prompts and docs, and click into `my-invoice-app` to see your actual app code.

---

## Optional: VS Code Workspace File

If you want VS Code to remember this setup, save it as a workspace:

1. **File → Save Workspace As...**
2. Save the `.code-workspace` file in your `dev\` parent folder as something like `my-work.code-workspace`
3. Next time: open VS Code → **File → Open Workspace from File** → pick that file

This makes it easy to jump back in exactly where you left off.

---

## Common Mistakes

### "I built my app inside the apparition folder"

If your app ended up at `apparition\my-invoice-app\`, that's fixable:

1. Close VS Code
2. In File Explorer / Finder, move your app folder up one level so it sits *next to* apparition
3. Reopen VS Code at the parent folder
4. Update any paths in your git config if needed

### "I can only see one folder in VS Code"

You opened a specific subfolder instead of the parent. Fix: **File → Open Folder** and select one level up.

### "The AI keeps putting files in the wrong place"

At the start of your session, tell the agent:
> "My Apparition toolkit is at `C:\dev\apparition`. My app is at `C:\dev\my-invoice-app`. Please always write files to my app folder."

---

## Why This Matters

- **Apparition stays clean.** You can pull updates to the toolkit without messing up your app.
- **Your app has its own git history.** It connects to your own GitHub/GitLab/Bitbucket repo, not the Apparition repo.
- **VS Code agents work correctly.** When both folders are visible, Copilot can read the toolkit docs and write to your app at the same time.
- **You can build multiple apps.** Just add more sibling folders — one per app, all sharing the same toolkit.

---

## Your App Carries Apparition With It

When the build agent scaffolds your new app, it automatically copies the key Apparition docs, prompts, and agent files into a `.apparition/` folder inside your app:

```
my-invoice-app\
  .apparition\          ← snapshot of the toolkit, seeded at setup
    prompts\            ← master-prompt, auth-prompt, feature-prompt, etc.
    docs\               ← architecture, database, deployment reference
    agents\             ← agent files so Copilot works standalone
  SPEC.md
  docs\implementation.md
  Program.cs
  ...
```

**What this means for you:**

- Once your app is set up, you don't need to keep the `apparition` folder open in VS Code. The agents and prompts are already inside your app.
- Your `apparition` folder becomes optional reference material — keep it, archive it, or delete it. It doesn't matter.
- If you want to update your prompts with a newer version of Apparition, just copy the updated files from the toolkit into your app's `.apparition/` folder.

**A note on git:** The `.apparition/` folder is intentionally committed to your app's repo. It's documentation and context — not code — so it belongs in version control alongside your app.

---

## Your App's .gitignore

The build agent always generates a `.gitignore` file for your app. It includes standard .NET ignores plus a safety entry for the Apparition toolkit folder — so even if your `apparition/` folder is sitting nearby, it will never accidentally get committed to your app's repo.

The entry looks like this:
```gitignore
# Apparition toolkit — ignore if stored as a sibling folder
apparition/
```

This means you can store Apparition wherever is convenient (including inside your `dev\` parent folder) without worrying about it polluting your app's git history.

---

## Color-Code Your VS Code Windows

If you're building more than one app, or working across multiple monitors, VS Code windows can all look identical and it's easy to lose track of which is which.

The fix: give each project its own title bar color. It takes about 30 seconds and makes a real difference.

**The build agent will offer this during setup.** Just pick a color and it will create the `.vscode/settings.json` file automatically.

If you want to set it manually, create or edit `.vscode/settings.json` in your app root:

```json
{
  "workbench.colorCustomizations": {
    "titleBar.activeBackground": "#1a3a5c",
    "titleBar.inactiveBackground": "#122840",
    "titleBar.activeForeground": "#ffffff",
    "activityBar.background": "#1a3a5c"
  }
}
```

**Color options to choose from:**

| Color          | Background | Good for                      |
| -------------- | ---------- | ----------------------------- |
| 🔵 Ocean Blue   | `#1a3a5c`  | main project, production      |
| 🟢 Forest Green | `#1a4a2a`  | dev / staging                 |
| 🟣 Deep Purple  | `#3a1a5c`  | experimental / side project   |
| 🟠 Burnt Orange | `#5c2e00`  | second app                    |
| 🔴 Deep Red     | `#5c1a1a`  | live / production hotfix work |
| ⬛ Slate        | `#2a2a3a`  | minimal, no distraction       |

You can also open the **Command Palette** (`Ctrl+Shift+P` / `Cmd+Shift+P`) and search for **"Preferences: Color Theme"** to preview full themes, or search **"workbench.colorCustomizations"** in Settings to edit the colors in the GUI.
