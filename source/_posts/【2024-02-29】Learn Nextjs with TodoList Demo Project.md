---
title: Learn Nextjs with TodoList Demo Project
date: 2024-02-29 10:00:00
tags:
  - nextjs
  - prisma
  - sqlite
---

`nextjs` is a nodejs library that is a framework for building modern web apps from backend and frontend. Here I will share my experience of learning nextjs by coding a todo list demo project. Here is [my project repository](https://github.com/ys558/TodoList-demo-by-nextjs)

<!-- more -->

## create a nextjs project

We may create a floder for our project:

```bash
mkdir nextjs-demo-todo-list
cd todoList-demo-by-nextjs
npx create-next-app@latest .
```

## install dependencies

For our simple demo project, we need to install `prisma`, that is a open-source ORM for nodejs.

```bash
npm install prisma
```

and we just use `prisma` to generate a sqlite database file:

```bash
npx prisma init --datasouce-provider sqlite
```

## create the ORM model and migrate the db schema

in `prisma/schema.prisma` file, we may updated as below:

```prisma
model Todo {
  id String @id @default(uuid())
  title String
  complete Boolean
  createAt DateTime @default(now())
  updateAt DateTime @updatedAt
}
```

run the migrate command:

```bash
npx prisma migrate dev --name init
```

we may see the `prisma/migrations` floder, and see the sql file.

```sql
-- CreateTable
CREATE TABLE "Todo" (
    "id" TEXT NOT NULL PRIMARY KEY,
    "title" TEXT NOT NULL,
    "complete" BOOLEAN NOT NULL,
    "createAt" DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updateAt" DATETIME NOT NULL
);
```

## init the db seed data

we may refer to [this prisma official tutorial](https://www.prisma.io/docs/getting-started/quickstart#4-explore-how-to-send-queries-to-your-database-with-prisma-client) to generate db seed data.

```bash
touch prisma/seed.ts
# using ts-node to run seed.ts file:
npm i -D ts-node
```

`seed.ts`:

```jsx
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

async function main() {
  // generate the first todo data:
  await prisma.todo.create({
    data: {
      title: "Hello World",
      complete: false,
    },
  });
}

main()
  .then(async () => {
    await prisma.$disconnect();
  })
  .catch(async (e) => {
    console.error(e);
    await prisma.$disconnect();
    process.exit(1);
  });
```

in our project's script, we may adding the config in `tsconfig.json` file:

```json
"ts-node": {
  "compilerOptions": {
    "module": "CommonJS"
  }
}
```

and then run the seed script:

```bash
npx ts-node ./prisma/seed.ts
```

## startup the prisma UI page

the prisma UI page is a web page that can help us to manage the db schema and data. we may run the command. it's very easy to use.

```bash
npx prisma studio
```

it can be accessed by `http://localhost:5555`

## create the db instance file

we may create a `src/db.ts` file to init the db instance:
the boliplate code is from [this prisma official tutorial](https://www.prisma.io/docs/orm/more/help-and-troubleshooting/help-articles/nextjs-prisma-client-dev-practices#solution) :

```ts
import { PrismaClient } from "@prisma/client";

const prismaClientSingleton = () => {
  return new PrismaClient({ log: ["query"] });
};

declare global {
  var prisma: undefined | ReturnType<typeof prismaClientSingleton>;
}

export const prisma = globalThis.prisma ?? prismaClientSingleton();

if (process.env.NODE_ENV !== "production") globalThis.prisma = prisma;
```

## update the frontend pages

for the homepage, we may update the `src/page.tsx` file:

```jsx
import { prisma } from "@/db";
import Link from "next/link";
import TodoItem from "./components/TodoItem";
import { redirect } from "next/navigation";

const getTodos = () => prisma.todo.findMany();

// "use server" field might be the nextjs way to handle the server side logic.
const toggleItem = async (id: string, complete: boolean) => {
  "use server";
  await prisma.todo.update({
    where: {
      id,
    },
    data: {
      complete,
    },
  });
};

const deleteItem = async (id: string) => {
  "use server";
  await prisma.todo.delete({
    where: {
      id,
    },
  });
  // here refresh the page again, we may see that the item deleted.
  redirect("/");
};

const Home = async () => {
  // fetching the data from the db:
  const todos = await getTodos();

  return (
    <>
      <header className="flex justify-between items-center mb-4">
        <h1 className="text-2xl">Todo List</h1>
        <Link
          href="/new"
          className="border border-slate-300 text-slate-300 px-2 py-1 rounded hover:bg-slate-700 focus-within:bg-slate-700 ouline-none"
        >
          New
        </Link>
      </header>
      <ul className="pl-4">
        {todos.map(({ id, complete, title }) => (
          <TodoItem
            key={id}
            id={id}
            title={title}
            complete={complete}
            toggleItem={toggleItem}
            deleteItem={deleteItem}
          />
        ))}
      </ul>
    </>
  );
};

export default Home;
```

### nextjs rounter

as above code, we may see that nextjs provide the `Link` component to handle the rounter logic:

when we clik the Link, just like the `<a/>` label, just to the new page:

```jsx
import Link from "next/link";

<Link href="/new">New</Link>;
```

we may code the file`src/new/page.tsx` file:

```jsx
import { redirect } from "next/navigation";
import Link from "next/link";
import { prisma } from "@/db";

const createTodo = async (data: FormData) => {
  "use server";

  const title = data.get("title")?.valueOf();
  if (typeof title !== "string" || title.length === 0)
    throw new Error("Invalid title");

  await prisma?.todo.create({
    data: {
      title,
      complete: false,
    },
  });
  redirect("/");
};

const Page = () => {
  return (
    <>
      <header className="flex justify-between items-center mb-4">
        <h1 className="text-2xl">New</h1>
      </header>
      <form action={createTodo} className="flex gap-2 flex-col">
        <input
          type="text"
          name="title"
          className="border border-slate-300 bg-transparent rounded px-2 py-1 outline-none focus-within:border-slate-100"
        />
        <div className="flex gap-1 justify-end">
          <Link
            href=".."
            className="border border-slate-300 text-slate-300 px-2 py-1 rounded hover:bg-slate-700 focus-within:bg-slate-700 ouline-none"
          >
            Cancel
          </Link>
          <button
            type="submit"
            className="border border-slate-300 text-slate-300 px-2 py-1 rounded hover:bg-slate-700 focus-within:bg-slate-700 ouline-none"
          >
            Create
          </button>
        </div>
      </form>
    </>
  );
};

export default Page;
```

## TodoItem component

for the `TodoItem` component, we may update the `src/components/TodoItem.tsx` file:

```jsx
"use client";
// TodoItem function is the server side function, so we must mark "use client" field for this component
type TodoItemProps = {
  id: string,
  title: string,
  complete: boolean,
  toggleItem: (id: string, complete: boolean) => void,
  deleteItem: (id: string) => void,
};

const TodoItem = ({
  id,
  title,
  complete,
  toggleItem,
  deleteItem,
}: TodoItemProps) => {
  return (
    <li className="flex gap-1 items-center">
      {/* here using the call back function toggleItem to pass the id and value: */}
      <input
        id={id}
        type="checkbox"
        className="cursor-pointer peer"
        defaultChecked={complete}
        onChange={(e) => toggleItem(id, e.target.checked)}
      />
      <label htmlFor={id} className="peer-checked:line-through">
        {title}
      </label>
      <div
        className="cursor-pointer rounded-full"
        onClick={() => deleteItem(id)}
      >
        ❌
      </div>
    </li>
  );
};

export default TodoItem;
```
