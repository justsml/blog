---
layout: post
title:  "Beautiful Data & Models"
date:   2015-09-22
modified:   2017-04-29
categories: programming
tags: [programming, patterns, models, source code, organization]
image:
  feature: abstract-5.jpg
  credit:
---

## The topic we'll examine is deceptively simple & subtle: Naming

> While covered in exhausting detail before, I will try to focus on a few example situations, each with more detail & code. You probably don't need to read this if you know what `Boyce Codd` is about.

> Recommended reading includes:
>
1. Book: Code Complete
2. [http://phlonx.com/resources/nf3/](http://phlonx.com/resources/nf3/)
3. [Database normalization](https://en.wikipedia.org/wiki/Database_normalization)


## The Problem - by Example

Have you ever designed a `data model` (including in code, Sql, or even Excel worksheets)?

Does the following look familiar?

```
*** Schema anti-pattern - don't do this at home ***
* User
  - id
  - avatarUrl
  - email
  - passwordHash
* Lead 
  - id
  - name
  - email
* Agent
  - id
  - primaryPhoto
  - agentName
  - agentEmail
  - agentPhoneMain
  - agentEmailPrimary
  - agentPhonePrimary
  - agentAddressLine1
  - agentCompanyName
  - agentCompanyAddress
  - *userEmail* - 'Pointer' to User table ^^^
```

Why is naming a field `agentEmailPrimary` the worst?

For starters, you are **not** creating an entirely new object unto the universe. Clearly you meant `email.` 
Over-specificity has many costs:

- Locked into highly specific name, means `agentEmailPrimary` probably make your views and related code **0% reusable**, and featuring annoyingly recurring bugs like:
- Data not syncing between tables (not obvious if `user.email` ever needs to propagate to `agent.agentEmail` or vice-versa - nevermind complexity of manually implementing where & how to enforce this 'logic')
- Validation rules/logic are likely duplicated & inconsitent.
- Increasingly, your project will resemble a shaky Jenga tower.
- Fragility piles up with every single new file, as an extremely high attention to detail is required for even trivial changes

I know, you probably feel something like...

![fuck this][fuck_this]


## A Solution


```
// Dan's Recommended Schema Consolidation:

User
  - id
  - role: ['agent', 'lead', 'admin']
  - name
  - phone
  - address
  - email
  - passwordHash
  - company
    - name
    - address

```

I removed the `Lead` and `Agent` table, as they didn't contain fields which were uniquely their own.

All changes were made with these general ideas in mind:

1. Eliminate unessesary tables. If you have a few dozen tables, this step is mandatory.
  1. Try merge related tables. All our example tables were essentially different generalizations of identity.
  1. Delete redundant data collection (e.g. remove `ActivityLogs` table if it's replaced by Google Analytics)
1. Keep **all field/key names** to a **single word/noun/pro-noun**.
  1. There is **no such thing** as `Agent.agentEmail` or `agentPhonePrimary`. Period.
  1. By using Highly Specific Names, you cast-in-stone a specific level of `code-reusability` and `durability,` well, specifically none.
  1. Don't think you are doing yourself any favors with crap like this `User.profileSummaryEmail.` Create a new table, call it `Profiles` and include an `email` field.




![teamwork][teamwork]

[schema_refactor]: https://res.cloudinary.com/ddd/image/upload/bldg-collapse__wsZKhIc_kafcha.gif
[not_a_fan]: https://res.cloudinary.com/ddd/image/upload/timeout-expired.gif
[teamwork]: https://res.cloudinary.com/ddd/image/upload/teamwork__tumblr_n2df80cPZa1s373hwo1_400_ghv4xn.gif
[fuck_this]: https://res.cloudinary.com/ddd/image/upload/panda-rampage__tumblr_nq7srwTXqr1stn6klo1_500_gm2som.gif
[new_feature]: https://res.cloudinary.com/ddd/image/upload/simba-toss-error.gif
[drinking]: http://res.cloudinary.com/ddd/image/upload/v1442175801/system-maint-anon.gif
[cat_outfit]: http://res.cloudinary.com/ddd/image/upload/v1441143858/cat-bee-fail.gif
[cat_loops]: http://res.cloudinary.com/ddd/image/upload/v1441143869/cat-loops.gif
[cat_bowl]: http://res.cloudinary.com/ddd/image/upload/v1441143883/kitten_bowl.gif
[cat_wtf]: http://res.cloudinary.com/ddd/image/upload/v1441143878/cat-wtf.gif
[endless_loop]: http://res.cloudinary.com/ddd/image/upload/v1441143881/endless-loop.gif

