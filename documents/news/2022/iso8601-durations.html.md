---
layout: blog.html.ejs
title: Support for ISO8601 duraton strings
publicationDate: June 13, 2022
blogtag: news
---

Included in the ISO8601 date standard is a string format for specifying time durations.  Such as _PT10S_ means a duration of _10 seconds_, and _P1YT10H_ means _1 year 10 hours_.

These duration strings are widely used and should be supported.  This validation is not included in the supporting package, and was implemented directly by using the `iso8601-duration` package.


