# Forbidden Brownie II — Writeup

**Category:** Web · **Points:** 250

## TL;DR

`/staff/notes.php?category=` has the same raw SQL injection as Forbidden Brownie I, except this time `UNION` and `SELECT` get quietly stripped out of your payload before it hits the database. We haven't looked at how that filter works ourselves, we only noticed the pattern by sending various payloads at it and deduced that sending the word to be filtered, along with a copy of itself in between, will produce exactly the result we want.

## Recon

Checked out the `notes.php` page of the staff area — all the same data from Pixels Canteen with the category tabs. There were some hints that lookups have been filtered since "last month's incident," meaning something has definitely changed.

## Step 1 — Reuse the old payload

Used the exact UNION SELECT payload from the previous challenge:

```
notes.php?category=Burgers' UNION SELECT id,name,category,secret_note,price FROM menu_items WHERE is_visible=0-- -
```

The result was "No prep notes found for this category". The injection point definitely existed (as we did not receive a different error message or block page), something about the query itself just broke.
 
## Step 2 – Trial and error approach to find the pattern
 
Started reducing the payload and checking the components of the request one-by-one in order to understand what is getting filtered out. Sent many combinations of the payload (removing some words, shuffling words, altering their case) and checked whether they return "found" or "not found".
 
It did not take long to discover that any payload containing the actual words UNION or SELECT resulted in "not found", regardless of the case, while everything else in the payload (such as quotes, comments, or other keywords) passed through without any problem. Therefore, something specific was removing these two words from the query before it was executed.
 
## Step 3 – Double-write bypass approach used in this case

Once it became clear that it was only these two terms that were being stripped out, the conventional method of getting around single-pass filtering attacks was tried: include the target word in-between a copy of itself, such that after the filter stripped away the inner term, what remained would reconstruct the word.
 
```
notes.php?category=Burgers' UNIUNIONON SELSELECTECT id,name,category,secret_note,price FROM menu_items WHERE is_visible=0-- -
```
 
The `UNIUNIONON` term becomes `UNION` once the middle `UNION` term is stripped away, while the `SELSELECTECT` term becomes `SELECT` similarly. Once the input had been modified by the filter to include the payload, the SQL query that was actually run is identical to the one from the previous challenge. As expected, the response included "Forbidden Brownie" along with the flag.
 
## Root cause
 
The filtering performed only a single scan for the terms and did not verify the output after performing any replacements, making it vulnerable to the inclusion of a duplicate copy of the targeted word within the input string. Filters that need to protect themselves against such an attack must continually rescan until no more changes occur.