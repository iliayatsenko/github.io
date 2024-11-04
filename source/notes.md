---
title: Small random notes
date: 2024-09-11
layout: page
comments: true
---

(Use Ctrl+F to find something)

<br/>

<details>
   <summary>

##### Run hexo in a docker container

   </summary>

```bash
    docker run  -it -v $(pwd):/var/www/blog -p 4000:4000 iliayatsenko1708/myimages:hexo bash 
```

</details>

---

<br/>

<details>
   <summary>

##### Encodings

   </summary>

Simplified extremely:

- Computers operate on numbers only, so in order to represent chars, some mapping is needed. Basically, encoding means this mapping. 
- **ASCII** (American Standard Code for Information Interchange) - mapping from 1-byte numbers to characters, can represent up to 255 chars (but uses only first 128). Covers all English letters, numbers, punctuation and some special chars.
- **ANSI code pages** - standardized set of encodings which extend ASCII. They represent different non-English characters using next 128 numbers. Example: Windows-1251.
- **Unicode** - next level of abstraction. Aims to represent all possible chars by mapping not directly to numbers, but to special codes (code points).
- **UTF-8, UTF-16 etc** - Unicode encodings. Control how code points are mapped to bytes. UTF-16 encodes each code point using from 2 to 4 bytes. It isn't backward compatible with ASCII, thus not very popular. Instead, UTF-8 uses from 1 to 4 bytes, and is compatible with ASCII (encodes first 128 chars the same way as ASCII).

Links:

- https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/
- https://csharpindepth.com/Articles/Unicode


</details>

---

<br/>

<details>
   <summary>

##### VSCode Hotkeys

   </summary>

open/hide sidebar: cmd b

file exprlorer: cmd shift e

open file in preview: space

open in new tab: cmd down

last edit location: cmd shift backspace

last location: ctlr - (back), ctrl shift - (forward)

reveal in explorer: shift cmd w

terminal: cmd j

latest files: ctrl tab

expand selection: cmd d

outline: shift cmd o

select all occurences of symbol: shift cmd l

double line: alt shift down/up

move line: alt down/up

add cursor: alt cmd down/up

focus on breadcrumbs: shift cmd dot

search for implementations/definitions: cmd/alt f12

goto matching bracket: cmd shift \

rename symbol: f2

see problems: shift cmd m

search for symbol: cmd t

search for symbol in editor: shift cmd o

search references to a symbol: shift f12

focus on open editors: cmd k e

list all open editors: cmd alt tab

column selection: shift alt (hold)

fold/unfold: cmd alt [/]

focus on next/prev part of screen: f6/shift f6

go to next/prev difference f7/shift f7

pin tab: cmd k shift enter

goto bracket: shift cmd \

goto next/prev change in file: alt f5 / shift alt f5

focus on peek editor: cmd k f2

open recent project: ctrl r

run and debug: cmd shift d

focus on next/prev editor/terminal group: cmd shift [/]

close other editors in group: cmd alt t


</details>


<!--- Template:

---

<br/>

<details>
   <summary>

##### Title

   </summary>

Content

</details>
-->

