# nextjs-13

## First
This project is my REACTION to the video ⬇️

<div>
  <a href="https://www.youtube.com/watch?v=6aP9nyTcd44"><img src="https://img.youtube.com/vi/6aP9nyTcd44/0.jpg" alt="IMAGE ALT TEXT"></a>
</div>

### Ref 
https://nextjs.org/blog/next-13

## App directory
### app/page.tsx
```js
function Home() {
  return <div> I am the homepage </div>
}
```
- Delete `index.tsx` to able to use new style home page 

This will auto create a file called `layout.tsx` this file is a handy way to share different UI component in sort of hierachical fashion way.

The pros: preverse the state and prevent unnecessarily re-rendering 

### app/layout.tsx

🧠 The `layout.tsx` is the way you define the layout for parent and childrent pages.

You can always overide this.

```js
export default function RootLayout({children})
return (
  <html>
    <head></head>
    <body> 
      <Header />
      {children}
     </body>
  </html>
)
```

### app/Header.tsx
```js
function Header() {
  return (
    <header> 
      <Link href="/">This is a Header </Link>
    </header>
  )
}
```
## Create another page 
### app/todos/page.tsx
```js
function Todos() {
  return (
    <div>
      <TodosList />
    </div>
  )
}
```
Access: localhost:3000/todos

🟡 All components are inside the `app` is server component 

### app/todos/TodosList.tsx
```js

const fetchTodos = async () => {
  const res = await fetch(`lol.me`)
  const todos: Todo[] = await res.json();
  console.log(`Todos`, todos) // 🔴 This one ONLY in terminal coz this component belongs to server side 
  return todos;
}

function TodosList() {
  const todos = await fetchTodos()

  return (
    {
      todos.map((todo)=>(
        <p key={todo.id}>
          <Link href={`/todos/${todo.id}`}> Todo: {todo.id} </Link>
        </p>
      ))
    }
  )
}
```

This is the typescript way ⏬
### typing.d.ts
```js
export type Todo = {
  userId: number;
  id: number;
  title: string;
  completed: boolean;
}
```

## Dynamic routes 
To prevent 404 for item details page 

### app/todos/[todoId]/page.tsx
```js
type PageProps = {
  params: {
    todoId: string;
  }
}

const fetchTodo = async (todoId: string) => {
  // const res = await fetch(`lol.me/${todoId}`))
  // update fetch api using cache 
  // const res = await fetch(`lol.me/${todoId}`), { cache: 'force-cache' })
  // no-cache: meaing sever side rendering 
  // force-cache: static generation 

  const res = await fetch(`lol.me/${todoId}`), { cache: { revalidate: 60 } })
  const todo: Todo = await res.json()
  return todo;
}

async function TodoPage({ params: { todoId }}: PageProps) {
  const todo = await fetchTodo(todoId)

  if (!todo.id) {
    return notFound()
  }

  return (
    <div>
      <p> {todo.id}: {todo.title} </p>
    </div>
  )
}

```

## Terms 
### Server side rendering 
Meaning async loading, you are waiting while geting data from other service then render it 
### Static rendering 
Meaning you are already had a static website 


### Render in advange 
We will pre-initialize the page at build time.
When you pack the website to deploy we will pre-render first (10) item details by using func `generateStaticParams`

Remember this function only run at build time
```js
export async function generateStaticParams() {
  const res = await fetch(`lol.me`)
  const todos: Todo[] = await res.json();
  /*
    We need return data in this format 
    [{todoId: '1'}, {todoId: `2`}, ...]
  */
 const trimmedTodos = todo.splice(0, 10);
 return trimmedTodos.map((todo) => ({
  todoId: todo.id.toString()
 }))
  
}
```

Ok, if we access page out of 10
- First, it will server load 
- Second, it cache the page for next time 


## dynamicParams

Ref: https://nextjs.org/docs/app/api-reference/file-conventions/
route-segment-config

```js
export const dynamicParams = true // true | false,
```

- true (default): Dynamic segments not included in generateStaticParams are generated on demand.
- false: Dynamic segments not included in generateStaticParams will return a 404.


### app/todos/[todoId]/not-found.tsx

```js
function NotFound() {
  return <div> Whoops! </div>
}
```

### app/todos/layout.tsx

```js
export default function RootLayout() {
  return (
    <main>
      <TodoList />
      /* 🔴 the children is the rest of the page */
      <div> {children} </div>
    </main>
  )
}
```

Replace app/todos/page.tsx

```js
function Todos() {
  return (
    <div>
      <h1> This is where the item detail will be render </h1>
    </div>
  )
}
```

🔴 With this `layout.tsx` 
we will see a list 1... n to-do list on the left and this text 
`This is where the item detail will be render` from `page.tsx` from the right 

WHEN tap any item on the LEFT the text ~~This is where the item detail will be render~~ will be replaced by the to-do-detail


we always keep the todolist only render the item details next to the list, meaning the item list will not re-render.
Layout will devide by we-want-section, boosting performance 




### app/search/page.tsx

```js
function Search() {
  return <div> Search </div>
}
```

Access: localhost:3000/search


### app/search/layout.tsx

```js
export default function RootLayout (
  return (
    <main>
      <h1> Search </h1>
      <div className="flex-1 pl-5">
        <Search />
        // the rest of the page is children
        <div> {children} </div>
      </div>
    </main>
  )
)
```

### app/search/Search.tsx

```js
function Search() {
  return (
    <h1> Search component </h1>
  )
}
```

### app/search/page.tsx

```js

function Search() {
  return <h1> Search </h1>
}

```
Access: localhost:3000/search 

### app/search/layout.tsx

```js
export default function RootLayout {
  return (
    <main>
      <div>
        <h1> Search </h1>
      </div>
      <div>
      </div>
        <Search />
        <div> {children} </div>
    </main>
  )
}

```

With this layout text `Search` in `page.tsx` will be replace when hit search btn. Then access `localhost:3000/search/halamhandsome`

### app/search/Search.tsx

Apply client component 

```js

`use client`

function Search() {
  const [search, setSearch] = useState("")
  const router = useRouter()

  const handleSearch = async (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setSearch("")
    router.push(`/search/${search}`)
  };
  return (
    <form onSubmi={handleSearch}>
      <input type="text" value={search} placeholder="Enter the Search term" onChane={(e) => setSearch(e.target.value)} />
      <button type="submit" className="btn"> Search </button>
    </form>
  )


}
```

### app/[searchTerm]/page.tsx

```js

type PapeProps = {
  params: {
    searchTerm: string;
  }
}

type SearchResult = {
  organic_results: [
    {
      position: number;
      title: string;
      link: string;
      thumbnail: string;
    }
  ]
}

const search = async (searchTerm: string) => {
  const res = await fetch(
    `https://me.lam/search?q=${searchTerm}`
  )
  throw new Error('WHOOPS something broke')
  const data: SearchResult = await res.json()
  return data;
}

async function SearchResults( { params: { searchTerm } }: PageProps ) {
  const searchResults = await search(searchTerm);
  return (
      <div>
        <p> You searched for: {searchTerm}</p>
        <ol>
        {
          searchResults.organic_results.map((result)=>(
            <li key={result.position}>
              <p> {result.title} </p>
            </li>
          ))
        }
        </ol>
      </div> 
    )
}

```

### app/search/[searchTerm]/loading.tsx

```js

function Loading() {
  return <div> Loading ... </div>
}
```

### app/search/[searchTerm]/error.tsx

```js

'use client';

export default function Error() {
  return (
    <div>
      <p> Something went wrong! </p>
      <button onClick={() => reset()}> Reset error boundary </button>
    </div>
  )
}
```

### app/search/head.tsx

```js

function Head() {
  return (
    <title>
      The search page
    </title>
  )
}
```

### Rout groups 
  You want seperate the user and admin without adding the `localhost/admin/list`

https://nextjs.org/docs/app/building-your-application/routing/route-groups

Example:
/(user)/search
/(user)/todo
/(admin)/develop

### Fallback 

```js 

function Home() {
  return (
    <div>
      <Suspense fallback=(<p> Loading ... </p>)>
      <TodoList />
      </Suspense>
    </div>
  )
}
```


### Page and layout 
https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts

### When to use Server and Client components?
https://nextjs.org/docs/getting-started/react-essentials#when-to-use-server-and-client-components

Client: Need browser api, statefullness (hook, useState)

<p><img type="separator" height=8px width="100%" src="https://github.com/HaLamUs/nft-drop/blob/main/assets/aqua.png"></p>

## Author

This repo was developed by [@lamha](https://github.com/HaLamUs). 
Follow or connect with me on [my LinkedIn](https://www.linkedin.com/in/lamhacs). 

## License
