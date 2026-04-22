---
title: Writing with Chirpy (Media and Typography)
date: 2026-04-22 16:40:00 +0300
description: Practical notes for using media, typography, code blocks, and prompt boxes in Chirpy posts.
categories: [Blog, Tutorial]
tags: [chirpy, markdown, media, typography]
image:
  path: /avatar.jpeg
  alt: Portfolio profile photo
---

This post is a short summary of the Chirpy documentation sections I use most often.

## Headings and Text

### Subheading Example

Regular paragraph text goes here. You can use **bold**, *italic*, and `inline code` where needed.

## Prompt Boxes

> This is a tip prompt.
{: .prompt-tip }

> This is an info prompt.
{: .prompt-info }

> This is a warning prompt.
{: .prompt-warning }

## Code Block

```yaml
categories: [Portfolio, Projects]
tags: [api, backend]
```
{: file="post-front-matter.yml" }

## Image Usage

![Profile image](/avatar.jpeg){: w="640" h="640" }
_A simple image caption example._

## Social Media Video Embed

{% include embed/youtube.html id='H-B46URT4mg' %}

## Mermaid Example

```mermaid
graph LR
  A[Idea] --> B[Design]
  B --> C[Development]
  C --> D[Release]
```

These examples provide a quick starting point when writing new posts.
