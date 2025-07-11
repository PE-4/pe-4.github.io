---
layout: post
title: "2,432 live secrets on the Chrome Web Store"
subtitle: "~ AKA, Analyzing 100K Chrome extensions for live secrets ~"
date: 2025-7-11
author: pe-4
---

# Introduction

I've seen a lot of great research from truffle security and others focused on scanning public resources like github repos, docker images, and even LLM training data for exposed secrets. But I started thinking: has anyone done this for browser extensions? I chose to scan the Chrome Web Store, since chromium based browsers are the most popular.

# Numbers
- **2,432 Live Secrets** were discovered.
- **36M+ users** using extensions containing live secrets.
## Top 5
- **Infura**: 242
- **OpenAI**: 132
- **Alchemy**: 67
- **AWS**: 62
- **OpenWeather**: 55

# Scraping Secrets
I didn't feel like writing a full on scraper for the Chrome Web Store turns out, someone already did the hard work. I grabbed a list of extensions from [DebugBear's chrome-extension-list](https://github.com/DebugBear/chrome-extension-list). It had a big enough sample size, so I just ran with it.

Then I found this [handy little script](https://gist.github.com/ckuethe/fb6d972ecfc590c2f9b8) that could download extensions by ID. I skidded it, made some quick edits to read from the extension list I grabbed earlier, and downloaded about 100K extensions into a folder called `extensions`.

After that, I used [TruffleHog](https://github.com/trufflesecurity/trufflehog), which detects and verifies secrets (and also supports scanning `.zip` files) to scan the downloaded extensions:

`trufflehog filesystem extensions --archive-timeout=300s --json --only-verified`

Any creds it found were saved to a JSON file, along with the extension's user count and developer contact info.

## Notable Exposures

### GitHub Tokens
A ton of extensions exposed GitHub tokens with overprivileged scopes and personal access tokens, one of which had access to fairly large github repos (500+ stars).

<img width="1516" height="281" src="https://github.com/user-attachments/assets/6e31131b-bacc-4801-93e6-2806a7dee109" />

### Mailgun & Mailchimp API Keys
Used for email automation and marketing attackers could easily steal mailing lists and send out emails from the victims domain.

<img width="1575" height="276" src="https://github.com/user-attachments/assets/ab0b4c5c-2b09-4e31-bb70-9a799da43c2c" />
<img width="994" height="212" src="https://github.com/user-attachments/assets/d1b80c8b-5aae-46e3-9d0d-f0a532157bf6" />

### AWS Access Keys
I found several AWS access keys leaked in different extensions. A lot weren't really useful but some leaked user data, one with chat features, one that offered note taking, and another that handled screenshots. These keys weren't just sitting there, they were leaking access to S3 buckets containing user data and screenshots.

<img width="857" height="347" src="https://github.com/user-attachments/assets/36dce976-ebb6-4072-be4f-20a40f15e89e" />

### API keys for AI services
A few popular extensions with over 300,000 downloads had OpenAI keys along with a shit ton of smaller ones had hardcoded API keys to AI services, basically a infinite supply of API credits.

<img width="1531" height="926" src="https://github.com/user-attachments/assets/86e1fda2-6da2-4398-8d14-448c65c4cb02" />


### SSH & Private Keys
I found full private SSH/private keys inside some extensions. Why would someone hard code something like this? I have 0 clue.

<img width="1862" height="201" src="https://github.com/user-attachments/assets/a8c17982-ad66-4dea-a7e7-85e4dc7c1c77" />
<img width="1877" height="228" src="https://github.com/user-attachments/assets/f74dcce2-cde6-4e60-b793-cbbf3e17b9c0" />

# Final notes
I emailed as many developers as I could and got quite a few of the exposed secrets revoked. Some didn't respond, and a lot of extensions are still live with active keys.

If you're developing extensions: don't hardcode secrets. If you're using them: maybe double check what you've installed.

