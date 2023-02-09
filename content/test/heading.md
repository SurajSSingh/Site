+++
title = "Markdown Heading Test"
date = 2023-02-07
updated = 2023-02-07
draft=true
+++

Heading 1 using Subtext style [^subtext]
=================================================================

Heading 2 using Subtext style [^subtext]
-------------------------------------------------------------


# Heading 1 using ATX style [^atx]

## Heading 2 using ATX style [^atx]

### Heading 3 using ATX style [^atx]

#### Heading 4 using ATX style [^atx]

##### Heading 5 using ATX style [^atx]

###### Heading 6 using ATX style [^atx]

<h1> Heading 1 using direct HMTL </h1> 

<h2> Heading 2 using direct HMTL </h2>

<h3> Heading 3 using direct HMTL </h3>

<h4> Heading 4 using direct HMTL </h4>

<h5> Heading 5 using direct HMTL </h5>

<h6> Heading 6 using direct HMTL </h6>

[^subtext]: Subtext Style: Minimum one underline symbol ("=" [equal sign] for h1 and "-" [minus sign/hyphen] for h2), preceding line(s) become text (so long as previous line is textually not empty), max 3 spaces indent for both symbols and text (separately calculated), blank line after is optional

[^atx]: ATX Style: "#" * `N` (where `N` = 1..=6) (min 1 space/tab after #, max 3 space indent, trailing space ignored, blank line before and after optional), text must be one the same line; N determines \<h`N`>
