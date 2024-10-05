---
title: "Jira with Elements Connect plugin: dynamic field fetching data from other system"
date: 2023-05-01
tags: jira
---

We needed to introduce better cross-team cooperation, **bring all** the **stakeholders** and include **all data! into** a single place - **a ticket**. And well, yes we use Jira ticketing tool. I have seen some Jira and tickets approaches with some level of integrations and we wanted to do some extra work and have a real-time fetched data from another SSoT (Single source of Truth) system.

To be more specific, although it is not that important for code examples and snippets below, I believe it will feel more real case scenario that helps image whether something like this could be useful / relevant for you or not. We needed to have a **Jira fields that would query and provide real-time data** (really up-to-date), **from another system**, and show this data in a drop-down menu in Jira. Either an option to query through tens of thousands of objects to pick the proper one or "just" to show a smaller list with currently valid options to choose from. (query IPAM / DCIM system to fetch prefixes, vlans, IP addresses that currently exists in the firm and choose available team owners etc.)

We found that we'd need to reach for some plugin, paid one as no free option was (is?) capable of doing this. We tried several plugins and ended up hapyilly with Elements Connect that seemed to offer everything we required. Even though looking at Velocity syntax was a bit tricky at first as I have never worked on such a thing we managed to pull off everything users (customers) wanted.




