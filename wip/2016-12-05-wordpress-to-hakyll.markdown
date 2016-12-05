---
author: Andrew McCluskey
comments: true
date: 2016-12-05
layout: post
slug: wordpress-to-hakyll
title: 'Wordpress to Hakyll'
categories: Programming
tags: haskell hakyll blogging wordpress
---

- Motivation - not a fan of wordpress
    + too complicated for my needs
    + expensive hosting
    + dislike having to look at PHP
    + like simplicity of static site generation
    + MOAR HASKELL
    + Syntax highlighting was a real pain, and I did it wrong
  
- Stack new blag hakyll-template

- Rescuing content from wordpress - exit-wp
    + tags and categories need to be space separated
    + Emacs dired regex-replace
    
- Syntax highlighting
    + highlight.js
    + Choose the languages you care about, add to template
    
- Theming
