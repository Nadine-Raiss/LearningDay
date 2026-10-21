# GHAS Hands-on: Attendee Guide

Oct 21, 2026 · @Damian

## What this is

In 45 minutes you will take a deliberately insecure web shop (OWASP Juice Shop), switch on GitHub's three security scanners, watch GitHub refuse a leaked password and a bad code change, and fix a real SQL injection with one click. Everything happens in the browser on github.com; there is nothing to install and nothing to pay for.

You need: a personal github.com account (a work or enterprise-managed account will not be able to copy the repo), signed in, in a desktop browser. Keep this guide in one window and GitHub in another.

The repo you start from: [github.com/OzDoll/ghas-juice-shop-demo](https://github.com/OzDoll/ghas-juice-shop-demo)

## Step 0: Make your own copy (2 minutes)

1. Open [github.com/OzDoll/ghas-juice-shop-demo](https://github.com/OzDoll/ghas-juice-shop-demo).
2. Click the green **Use this template** button (top right), then **Create a new repository**.
3. **Owner**: your username. **Repository name**: `ghas-juice-shop`. Leave **Public** selected; public is what makes the security features free.
4. Click **Create repository**. After a few seconds you are looking at your own copy, with your username at the top left.

From here on, everything is in *your* repo. Nothing you do touches the original.

## Step 1: Switch on the scanners (7 minutes)

All switches are on one page. Some rows are on/off toggles, some have an **Enable** or **Set up** button. If a row already says On or Enabled, leave it.

1. In your repo, click the **Settings** tab (gear icon, far right of the tab row).
2. In the left column, under **Security and quality**, click **Advanced Security**.
3. **Dependency graph**: toggle **On**. This lets GitHub read the project's list of packages.
4. **Dependabot alerts**: toggle **On**. Alerts for packages with known security holes.
5. **Dependabot security updates**: toggle **On**. GitHub will open pull requests that upgrade those packages for you.
6. Scroll to **Code Security**. In the **CodeQL analysis** row, click **Set up**, then **Default**. A dialog summarises what will be scanned; change nothing and click **Enable CodeQL**. The first scan starts now and takes 2 to 5 minutes.
7. Make sure **Copilot Autofix** (same section) is on. It needs no Copilot subscription.
8. Scroll to **Secret Protection**. **Secret Protection** and **Push protection** are normally already on for public repos; if either shows **Enable**, click it. If you see extra rows such as **Validity checks**, turn them on too; if you do not see them, skip them.
9. If there is a **Save changes** button at the bottom, click it.

Check: click the **Actions** tab at the top. A run called **CodeQL** with a yellow dot is in progress. Do not wait for it; go to Step 2.

## Step 2: Watch GitHub refuse a token (7 minutes)

Push protection checks every change *before* it is saved. If it finds a real-looking password or key, the save does not happen. To see it, you need a real token, because it looks for real token shapes. You will make one with no permissions at all, so it is harmless.

**Make the token**

1. Open [github.com/settings/tokens](https://github.com/settings/tokens). (This is your account's settings, not the repo's.)
2. Click **Generate new token**, then **Generate new token (classic)**. Confirm your password if asked.
3. **Note**: anything. **Expiration**: 7 days. Under **Select scopes**, tick nothing. Scroll down, click **Generate token**.
4. Copy the string that starts with `ghp_`. You cannot see it again after leaving the page; if you lose it, make another.

**Try to save it into the code**

5. Back in your repo: **Code** tab, click the `routes` folder, click `login.ts`.
6. Click the pencil icon (top right of the file) to edit.
7. Click at the end of line 18, press Enter, and on the new line paste (with your token in place of the placeholder):

```
const secret = "ghp_PASTE_YOUR_TOKEN_HERE";
```

8. Click the green **Commit changes...** button. In the box, choose **Create a new branch for this commit and start a pull request**, name the branch `lab-secret`, click **Propose changes**.
9. Instead of a pull request you get a red banner: **Push protection** found a *GitHub Personal Access Token* in that file on that line, and offers a bypass with a reason.

That is the whole point: nothing was saved, the token never entered the project. Click the browser's back button and discard the edit. When you are done with the workshop, delete the token on the tokens page, or let it expire.

## Step 3: Read the alerts, let Copilot fix one (9 minutes)

Everything here is under your repo's **Security and quality** tab (shield icon). If **Code scanning** is still empty, your scan has not finished: do the first two parts and come back.

**Dependabot: outside packages**

1. **Security and quality** tab, then **Dependabot** in the left column. A list of alerts with severity badges.
2. Open **Command Injection in marsdb**. On the right there is no fixed version to upgrade to. In real life you would accept the risk, swap the package, or write the feature yourself. Go back.
3. Open **Verification Bypass in jsonwebtoken**. This one has a fix. If a **Review security update** button is showing, click it: that is a pull request Dependabot wrote on its own. Go back.
4. In the search box above the list, replace the text with `is:open has:patch` and press Enter. This is the list to start with: everything that disappears with an upgrade.

**Secret scanning: leaked keys**

5. Left column, **Secret scanning**. Open the **Google API Key** alert. It shows the file (`secrets.txt`) and line.
6. Click **Verify secret**. GitHub asks Google whether the key still works; it reports inactive.
7. Click **Close as** (top right), choose **Revoked**. Secret alerts never close on their own, even if you delete the file: the key was already public. You revoke it at the provider, then close the alert yourself.

**Code scanning: the code itself**

8. Left column, **Code scanning**. Open the alert **Database query built from user-controlled sources** whose location is `routes/search.ts`.
9. This is SQL injection: a user can type database commands into the search box and the code passes them straight to the database. CWE-89 is the catalogue number for this kind of hole.
10. Click **Show paths**. CodeQL did not search for a bad word; it followed the data from the user's input to the database query and found nothing cleaning it on the way.
11. Click **Show more** for a plain-language explanation and the recommended fix.
12. Click **Generate fix** (top right). After 30 to 60 seconds a suggested code change appears. This is Copilot Autofix: it read the alert and the surrounding code and wrote the correction.
13. Click **Commit to a new branch**, leave the defaults, click **Commit change**. GitHub creates a branch and a draft pull request with the fix. You do not need to merge it today; if you did, the alert would close by itself because the problem would be gone from the code.

## Step 4: Break the login, get blocked, fix it with one click (13 minutes)

You will put a SQL injection into the login code on a side branch, ask to merge it, and watch CodeQL catch it, a rule refuse the merge, and Copilot offer the fix. Do the pull request first so its scan starts, then make the rule while it runs.

**Break the login on a branch**

1. **Code** tab, `routes` folder, `login.ts`, pencil icon.
2. Find line 36, which starts with `models.sequelize.query(`. Lines 36 to 46 are one block ending with a line that is just `)`. Select all eleven lines and delete them.
3. On the empty line 36, paste this as one line:

```
models.sequelize.query(`SELECT * FROM Users WHERE email = '${req.body.email || ''}' AND password = '${security.hash(req.body.password || '')}' AND deletedAt IS NULL`, { model: UserModel, plain: true })
```

4. Check that line 37 now starts with `.then(`. If not, undo (Ctrl+Z / Cmd+Z) and try again.
5. **Commit changes...**, choose **Create a new branch for this commit and start a pull request**, name it `lab3/code-scanning-vulnerability`, click **Propose changes**.
6. On the **Open a pull request** page, click **Create pull request**. (Optional: click the Copilot icon in the description box first and let it write the summary.)
7. At the bottom of the pull request, a checks box shows **CodeQL** running. It takes 3 to 5 minutes. Do not wait; go to step 8.

**Add the rule while it scans**

8. **Settings** tab, then in the left column under **Code, planning, and automation** click **Rulesets**.
9. **New ruleset**, then **New branch ruleset**.
10. **Ruleset Name**: `block-vulnerable-code`. **Enforcement status**: change to **Active**.
11. Under **Target branches**, click **Add target**, choose **Include default branch**.
12. Scroll down and tick **Require code scanning results**. Under **Required tools and alert thresholds**, click **Add tool** and choose **CodeQL**. Set **Alerts** to **Errors** and **Security alerts** to **High or higher**.
13. Scroll to the bottom, click **Create**.

**Watch it work**

14. **Pull requests** tab, open your `lab3/code-scanning-vulnerability` pull request.
15. When the scan finishes: the **CodeQL** check has a red cross, and the **Merge pull request** button is grey, with a note that code scanning results are required. Nobody can merge this by accident.
16. Scroll up in the **Conversation** tab. The **github-advanced-security** bot has commented on line 36 describing the SQL injection, with a **Suggested changeset** from **Copilot Autofix** underneath restoring the safe query. (Give it a minute after the scan if it is not there yet.)
17. Click **Commit suggestion**, then **Commit changes**. The scan runs again on the fixed code; when it passes, the merge button turns green.

That is the whole loop: found, blocked, fixed, inside one pull request. You do not have to merge it.

## Words used above

| Word | Meaning |
| --- | --- |
| Repository, repo | A project folder on GitHub: the code, its history, its settings |
| main | The main line of the code, the version treated as real |
| Branch | A side copy of the code where you change things without touching `main` |
| Commit | Saving a change, with a short message |
| Pull request (PR) | A request to bring a branch's changes into `main`. A page where the change is shown, checked by robots, and merged or closed |
| Merge | Accepting a pull request |
| Alert | One security problem a scanner found |
| Dependabot | Scanner for outside packages the code uses; knows which versions have known holes and can open PRs to upgrade them |
| CodeQL, code scanning | Scanner that reads the code itself and finds holes like SQL injection. Code scanning is the feature; CodeQL is the engine |
| Secret scanning | Scanner for passwords, keys and tokens left in the code |
| Push protection | Refuses to save a change that contains a password or key |
| Token (PAT) | A long random string that works like a password for GitHub |
| Ruleset | A rule on `main`, such as "nothing merges until the code scan is clean" |
| Copilot Autofix | GitHub's AI that writes the fix for a code scanning alert; no Copilot subscription needed |
| Actions | GitHub's robot that runs jobs like the CodeQL scan |
| Check | A job's result on a pull request: green tick or red cross |

## If you want more

All of this is free on your public copy, so keep going after the session.

- The original 8-lab workshop this session was cut from, by GitHub's own team: [githubuniverseworkshops/juice-shop-ghas-workshop](https://github.com/githubuniverseworkshops/juice-shop-ghas-workshop). Lab 4 adds a check that blocks vulnerable packages in pull requests; Lab 8 lets you define your own secret patterns. Where those labs say "Code security", the page is now called **Advanced Security**.
- Pick a second alert from Step 3 (an XSS or path-traversal one) and fix it the same way.
- [About GitHub Advanced Security](https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security) and the [GitHub Security Learning Pathway](https://resources.github.com/learn/pathways/security/).

In a company, the same alerts from hundreds of repositories roll up into an organization-wide Security Overview (which teams have the most open problems, which repos have the scanners on, how often push protection was bypassed). That needs an organization on GitHub's enterprise plan, which is why we did not click into it today; what a developer does stays the same.
