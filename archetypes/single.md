---
id: "{{ sha1 .Name }}"
title: "{{ replace .Name "-" " " | title }}"
description: ""
date: {{ .Date }}
images:
    - "/images/og_image.png"
nometadata: true
notags: true
noshare: true
nocomments: true
---

