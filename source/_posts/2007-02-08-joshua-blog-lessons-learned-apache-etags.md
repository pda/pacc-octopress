---
title: "joshua's blog: lessons learned: apache etags"
date: 2007-02-08 13:59:57 +11:00
link: http://joshua.schachter.org/2006/11/apache-etags.html
tags: apache http optimization performance etag scalability
layout: link
---
Apache-generated etags for objects mirrored across multiple backend servers differ because the disk inode is a factor in the checksum.
