---
title: Learn Web3
date: 2024-03-02 10:00:00
tags:
    - web3
    - eth
---

this article is about web3 learning note. I will be keeping update it for my whole web3 learning until I find the web3 work.

<!-- more -->

```jsx
import { Suspense, use } from "react";

const fetchData = async () => {
  const res = await fetch('https://jsonplaceholder.typicode.com/todos');
  const data = await res.json();
  return data;
}

const JokeItem = () => {
  const items = use(fetchData())
    return items.map(({ id, title }) => <p key={id}>{title}</p>)
}

const Joke = () => {
  return <Suspense fallback={<h2>Loading...</h2>}>
    <JokeItem />
  </Suspense>;
}

export { Joke as UseExample1 };
```

