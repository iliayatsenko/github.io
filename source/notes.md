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

###### Run hexo in a docker container

   </summary>

```bash
    docker run  -it -v $(pwd):/var/www/blog -p 4000:4000 iliayatsenko1708/myimages:hexo bash 
```

</details>

<br/>

<details>
   <summary>

###### Encodings

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


<!--- Template:

<br/>

<details>
   <summary>

###### Title

   </summary>

Content

</details>
-->

