---
title: "2023 New Year Postcard"
date: 2023-12-27T22:57:12+09:00
draft: false
tags: []
categories: []
isCJKLanguage: true
toc:
  enable: true
  auto: true
---


# Countdown <div id="2023NewYearCountdown"></div> 

This page will be hidden until new year coming!

本页面将会在2024新年到来的时候更新内容喵~


<script type="module">
  import { formatDistanceStrict } from "https://cdn.jsdelivr.net/npm/date-fns/formatDistanceStrict.mjs";
  const cditem = document.getElementById("2023NewYearCountdown"), new_year_2024 = new Date("2024-1-1")
  setInterval(()=>{
    cditem.innerText = formatDistanceStrict(new_year_2024, Date.now())
  }, 1000)
</script>
