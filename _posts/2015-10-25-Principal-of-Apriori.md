---
layout: post
title:  "Principal of Aprirori"
date:   2015-10-25 20:16:22
categories: Algorithm
author: xxiieao
---

* TOC
{:toc}

---

# Overview

- DataMining
- Frequent item-set
- Bottom up approach

---

# Base Theory of Apriori

- If one item-set is *FREQUENT*, whose subsets are all *FREQUENT*.
- If an item-set is *INFREQUENT*, none of the item-sets contains that would be *Frequent*.

---

# Steps of Apriori

1. step1 counting and filtering candidate
2. step2 candidate generation
3. step3 return to step1, unless no or only one candidate left

---

# Step 1

| order_id   | item_name  |
| :--------: | :--------: |
| 1          | b          |
| 1          | c          |
| 2          | b          |
| 2          | e          |
| 3          | f          |


> counting
>
> b: 2, c: 1, e: 1, f: 1
>
> filted result
>
> b: 2

---

# Step 2

- Single item scenario

> items from step 1
>
> a: 10, b: 5, c: 8, e: 7
> 
> generate candidate from a, b, c, e
>
> [ab, ac, ae, bc, be, ce]

- Multi item scenario

> items from step 1
>
> abc: 4, abd: 5, acd: 3
> 
> generate candidate from abc, acd, ace
>
> [acde]