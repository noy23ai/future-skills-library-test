# Your Future Skills library

This repository is a plugin marketplace for [Claude](https://claude.ai) holding the
skills you saved on [Future Skills](https://marketplace.future.security).

It is written by the Future Skills GitHub App, which can only write to this one
repository. Uninstall the app to stop that; delete this repository to remove it
entirely. Neither affects your Library on the site.

## Using it

In Claude Desktop: **Customize → Plugins → Add marketplace**, and give it this
repository's URL. There are **two plugins** in here — install both:

- **My Skills** — the skills you saved. This is the one to press
  **Update** on after saving something; Claude does not check on its own.
- **Future Skill Finder** — lets you search Future Skills from inside a session
  and save what you find. The same for everybody, and it does not change when
  you save a skill, so updating it will not bring one in.

## What is in it

- **brainstorming** — scan: fail, trust: CAUTION
- **playwright-cli** — scan: fail, trust: CAUTION
- **imagegen-frontend-mobile** — scan: pass, trust: SAFE

## Checking what you got

`plugins/my-skills/FUTURE-SKILLS.json` records a digest of every skill written here,
with the scan verdict it carried at the time. It is a record you can verify, not
a lock — nothing refuses to install if it does not match.
