---
title: "将 Ghost 存储的博文导入到 Hugo 中"
date: 2021-06-22T02:31:40+09:00
tags: ["Tool", "Code", "Hugo", "Blog", "Ghost"]
categories: ["Tool"]
draft: false
isCJKLanguage: true
---

## 前置需求

本脚本使用 Node.js 写成，所以需要一个起码支持 ES6 module 的 Node 即可正常运行喵~

## 使用方式

1. 新建一个目录；
1. 将下列 [文件列表](#文件列表) 里的文件变成磁盘上的文件；
3. 进入该目录， 执行 `npm i` 安装依赖库；
4. 执行 `node .\index.mjs your-path-to\ghost.db [export_path]` 即可喵~

## 已有功能

* 自动生成 YAML 格式的 Front Matter； 
* 导出 Ghost 的 Tags 并写入 Front Matter；
* 自动格式化日期（时区是写死的东八区要注意喵！）
* 自动过滤标题转义符进行转义
* 按照 `YYYY-MM\slug.md` 的格式存储，不会一次性在目录下弄出大量文件也避免文件名冲突喵~

## 文件列表

index.mjs
```js
import sqlite3 from "sqlite3";
import { open } from "sqlite";
import moment from "moment";
import process from "process";
import fs from "fs/promises";
import path from "path";
import { fileURLToPath } from "url";


if (process.argv.length < 3) {
    console.log(
        `Usage: ${process.argv[0]} ${process.argv[1]} sqlite_db.file [export_path]`
  );
  process.exit(1);
}

var content_path = "";

if (process.argv.length > 3) {
    content_path = process.argv[3];
} else {
  const __dirname = path.dirname(fileURLToPath(import.meta.url));
  content_path = path.join(__dirname, "content");
}

(async function main() {
  const db = await open({
    filename: process.argv[2],
    driver: sqlite3.cached.Database,
  });

  await fs.mkdir(content_path, { recursive: true });
  const tags = await db.all(`SELECT * from "tags"`);
  let tag_map = {};
  for (let tag of tags) {
    tag_map[tag.id] = tag.name;
  }

  const posts = await db.all(`SELECT * from "posts"`);

  const posts_tags = await db.all(`SELECT * from "posts_tags"`);
  let post_tag_map = {};
  for (let ptm of posts_tags) {
    if (ptm.post_id in post_tag_map) {
      post_tag_map[ptm.post_id].push(tag_map[ptm.tag_id]);
    } else {
      post_tag_map[ptm.post_id] = [tag_map[ptm.tag_id]];
    }
  }

  for (const post of posts) {
    const post_content = `---
title: "${post.title.replace(/\\/g, "\\\\")}"
date: ${moment(post.created_at).format(
      moment.HTML5_FMT.DATETIME_LOCAL_SECONDS
    )}+08:00
lastmod: ${moment(post.created_at).format(
      moment.HTML5_FMT.DATETIME_LOCAL_SECONDS
    )}+08:00
draft: ${post.status === "draft"}
isCJKLanguage: true
${
  post.id in post_tag_map
    ? 'tags:\n  - "' + post_tag_map[post.id].join('"\n  - "') + '"\n'
    : ""
}---

${post.markdown}`;

    const post_path = path.join(
      content_path,
      `${moment(post.created_at).format("YYYY-MM")}`
    );
    const post_file = path.join(post_path, `${post.slug}.md`);
    await fs.mkdir(post_path, { recursive: true });
    await fs.writeFile(post_file, post_content);
  }
})();

```

package.json
```JSON
{
  "name": "ghost-import-md",
  "version": "1.0.0",
  "description": "Import ghost blog posts to .md static files.",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "MIT",
  "dependencies": {
    "moment": "^2.0.0",
    "sqlite": "^4.0.23",
    "sqlite3": "^5.0.2"
  }
}

```