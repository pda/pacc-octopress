---
title: FriendFeed's schema-less MySQL datastore - Bret Taylor's blog
date: 2009-03-02 10:13:06 +11:00
link: http://bret.appspot.com/entry/how-friendfeed-uses-mysql
tags: architecture mysql scalability database schema friendfeed sharding
layout: link
---
All data stored opaque in a single sharded table, with a separate table created for each index. No assumption of index:data integrity - data returned from indices refiltered in userspace.
