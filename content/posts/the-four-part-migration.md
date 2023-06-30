---
layout: post
title: "The Four Part Migration"
date: 2023-05-11
description: If you've been writing relational database-backed applications for long enough, you've needed this.
tags:
- software development
- databases
---

If you've been writing relational database-backed applications for long enough, you've needed this.  The situation is common enough: You have a some kind of web application, you don't want to have any downtime, and you need to make a breaking change to your schema.

Breaking changes come in several forms: renaming a column, changing its type, or even just changing the format in which you store data (e.g. normalizing phone numbers), etc.  What these cases have in common is that once the change is made, the code that relied on the way that things were will no longer function.  For instance, when a column is renamed, code that's looking for the data under the old name won't work.  Code looking for data under the _new_ name won't function either until the rename is complete.

One approach would be to design your system to try the new column name first and fall back to the old column name in the case that the new one doesn't exist, but the ORMs I've used (ActiveRecord and Ecto) don't natively support this; they generally cause errors when a column you said exists doesn't.  Thus—for a while—we'll need both the old column (or table, or format, or whatever) to exist alongside the new column (or table, or format, or whatever).

For the remainder of this article, I'll be using the example of renaming a database table.  I hope the mapping is clear for other kinds of breaking changes and I will try to add some pointers where approaches may diverge.

Let's say that you've got a table named "inovices."  Whoops!  Somebody misspelled that!  Well it's in production now, but the team wants to rename this table to "invoices" so that it's less confusing.

## Part 1: Write Twice, Read Old

Fight the impulse to `ALTER TABLE inovices RENAME TO invoices`—this will break the running code!  Instead, create a new `invoices` table with an identical schema.  Now our old code can use the old, misspelled table **at the same time as** our new code can (theoretically) use the new, correctly spelled, table.

> But wait, these two tables don't have the same data in them!

Yes, that's a problem we'll need to fix.  The first step is to update our application such that when we add new rows to, or update the existing rows of, the old `inovices` table, we also write that data to the new `invoices` table.  This way, all data created or updated after this first step is deployed will be available in both tables.

<aside>
In the case of "renaming" tables, it's important that records across the two tables share primary keys.  Keys are how records in relational databases relate to one another, so it's important that when we swap out the new table for the old one, our relationships don't break!  If you're using your database to generate your primary keys, you'll likely want to store new rows in the old table first, returning their primary keys, and then specify those primary keys explicitly when storing those rows in the new table.
</aside>

Also, it should probably be intuitive here that our application, while _writing_ to both new and old tables, should only be _reading_ from the old (`inovices`) table.  That's the only table that has all our data in it!

Alright, once we've created the new table and updated our application to write twice but still read from the old source, we can now deploy this chunk of work.

## Part 2: Backfill

Ah yes, the glamour-less backfill task.  Now that Part 1 is deployed, our new table is being populated with new data, but it's still missing all the old data that the old `inovices` table has!  Time to fix that.

There's no cross-framework standard on how to write backfill tasks—and heck, you could just connect to the database directly to perform this operation—but the idea here is simply that for each record in the old table (or column or whatever) that does not yet exist in the new table (or column or whatever), we need to create a corresponding record in the new table (or column or whatever).  I trust that you can figure this out.

When this backfill task is finished running, the data inside the two tables should be identical.  And, since we're writing identical data to both, they should stay in-sync.

## Part 3: Read New, Stop Writing Old

This is probably the most gratifying step!  Here we update our application to forget that the old, misspelled `inovices` table ever existed.  We can read from the new—properly spelled—table because it's got all the same data in it, and we don't have to write to the old table anymore because we're not reading it anymore and never will again!

## Part 4: Delete Old

Once the deploy of Part 3 is complete, our application is no longer attempting to read or write to the old table.  Thus, we're free to drop it!  Run `DROP TABLE inovices` at your discretion.

## Conclusion

Admittedly, the Four Part Migration isn't very fun or glamorous.  However, in my experience this is the only safe way I've seen to make a breaking change to your schema without downtime (specifically looking at you, [blue/green deployments]).  I hope this post helps you solve your schema change conundrum!

I don't have comments on my blog, but if you have feedback, please reach out to [me on Mastodon/Fedi]!

[blue/green deployments]: https://docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/bluegreen-deployments.html
[me on Mastodon/Fedi]: https://fosstodon.org/@j3rn
