# GitHub Advanced Security sample demo

**CASA EMEA Learning Days, 21 October 2026**

This repository is a copy of [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/), a web shop written with security holes on purpose, used worldwide for security training. We use it to show what GitHub's three security scanners find and how a developer fixes it, all from the browser, in 45 minutes.

## What you will do

1. Switch on Dependabot, CodeQL code scanning and secret scanning
2. Watch push protection refuse a leaked token
3. Read one alert of each kind and let Copilot Autofix write a fix
4. Put a SQL injection into a pull request, get blocked by a ruleset, accept the one-click fix

## How to take part

- You need a personal github.com account (not an enterprise-managed one), signed in, in a desktop browser. Nothing to install.
- Click the green **Use this template** button at the top of this page, then **Create a new repository**. Keep it **Public**; that is what makes the security features free.
- Follow the [Attendee Guide](./ATTENDEE-GUIDE.md) in your copy. It has every click and takes about 45 minutes.

If you only have a work account, follow along on screen; the guide works the same afterwards on any personal account.

## Afterwards

- The full 8-lab workshop this demo was cut from, by GitHub's own team: [githubuniverseworkshops/juice-shop-ghas-workshop](https://github.com/githubuniverseworkshops/juice-shop-ghas-workshop). The lab files are still in [`_labs/`](./_labs/). Where they say "Code security" in Settings, the page is now called **Advanced Security**.
- [About GitHub Advanced Security](https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security)
- [GitHub Security Learning Pathway](https://resources.github.com/learn/pathways/security/)

## Credits

Based on the GitHub Universe workshop *Securing your code with GitHub Advanced Security* by [@joshjohanning](https://github.com/joshjohanning) and [@mickeygousset](https://github.com/mickeygousset), MIT licensed. Juice Shop is an OWASP project.
