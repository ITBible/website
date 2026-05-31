---
title: "How to Contribute"
description: "Learn how to contribute to our blog."
slug: contribute
showTableofContents: true
showhero: false
showwordcount: false
showdate: false
showreadingtime: false
showrelatedcontent: false
---

Something I've always wanted to do is allow people to share their experiences in the technology space and just help others out. This gives you the ability to grow your portfolio (to share with an employer), show others what you've been doing and just generally be involved.

## First time contributing
There is a bit of setup that you need to complete on your end to contribute.

### Fork the repository
Fork the repository. If you need more help please [see this article](https://docs.github.com/en/get-started/exploring-projects-on-github/contributing-to-a-project).
https://github.com/ITBible/website

### Create your Author Entry
You must create two different files for your author entry. 
Create data/authors/username.json and make sure to use your profile image from the community. In the below example I will create `/data/authors/wizardtux.json`.
```json
{
    "name": "Connor aka WizardTux",
    "image": "https://community.itbible.org/user_avatar/community.itbible.org/wizardtux/288/4_2.png",
    "headline": "I touch grass more now.",
    "bio": "I love technology but sometimes I need a break.",
    "links": [
        { "discord": "https://discord.gg/rMA5KapVyq" },
        { "facebook": "https://facebook.com/username" },
        { "github": "https://github.com/WizardTux" },
        { "tiktok": "https://tiktok.com/@WizardTux" },
        { "twitch": "https://twitch.tv/WizardTux" },
        { "x-twitter": "https://twitter.com/WizardTux" },
        { "youtube": "https://youtube.com/@ITBible" }
    ]
}
```
then create content/authors/username/_index.md (where username is your username from the first file) in this example I will create `content/authors/wizardtux/_index.md`.
```markdown
---
title: "Connor aka WizardTux"
---

I love technology but sometimes I need a break.
```

## Subsequent times contributing
Every article should be under blog or videos under the year, month (e.g. in 2026/05) then the folder containing your article should contain the day (e.g. 31_slug-of-your-file). While this isn't required it keeps the content organized.

Every article folder should contain a featured image with the name of the image file being "featured" this folder can also contain other images you use in your articles.
### Frontmatter Template
Every file should have "Frontmatter" at the beginning of each file. There are a lot of items that can go here to customize the look of your article but below is just a basic template that is used on similar articles here, if you would like to browse other articles you may pull frontmatter from them as well.
```markdown
---
title: '' # Title of the article
description: '' # Description of the content
summary: '' # Same as description above
categories: 
    - General # please use an existing category, we're trying to keep this limited
tags: [Tag1, Tag2] # please use a sensible list of tags here
date: 2026-05-31 # date that you want displayed on the article, this must not be in the future
slug: slug-from-your-folder-name
showTableOfContents: true
authors:
    - "yourusernamehere" # there must be at least one author, but you can have multiple listed here
    - "author2here"
---
```
Other options for Frontmatter can be found in `config/_default/params.toml` under the `article` section.