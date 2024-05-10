---
title: Nodejs SSR simple Demo
date: 2024-02-29 10:00:00
tags:
    - node
    - SSR
---

SSR is a technology that can render the html on the server side that can be convienient for SEO.
Here is a simple demo to show how to use `express` to implement SSR. 

<!-- more -->

## 1. Install

`touch ssr-demo-by-node` , `cd ssr-demo-by-node` run:

```bash
npm init -y
npm i express react react-dom wepback webpack-cli @babel/core @babel/preset-env @babel/preset-react babel/node @babel/presetts-react
```
## 2. Create a server

in the project folder create `server.js` file:

```js
const express = require('express');
const app = express();
const port = 3000;
  app.get('/', (req, res) => {
      res.send('Hello World!');
      res.end();
      console.log('Hello World!');
      console.log(req);
      console.log(res);
      console.log(req.url);

  }
  );
  app.listen(port, () => {
    console.log(`Example app listening at http://localhost:${port}`);
  }
  );
  module.exports = app;
  exports = module.exports = app;
```

