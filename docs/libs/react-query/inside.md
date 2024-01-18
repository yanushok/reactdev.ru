---
description: React Query изнутри
---

# React Query изнутри

## The QueryClient {#the-queryclient}
### A vessel that holds the cache {#a-vessel-that-holds-the-cache}
## QueryCache {#querycache}
## Query {#query}
## QueryObserver {#queryobserver}
### Active and inactive Queries {#active-and-inactive-queries}
## The complete picture {#the-complete-picture}
## From a component perspective {#from-a-component-perspective}







```ts title="pre-filtering" hl_lines="17-26"
type State = 'all' | 'open' | 'done'
type Todo = {
  id: number
  state: State
}
type Todos = ReadonlyArray<Todo>

const fetchTodos = async (state: State): Promise<Todos> => {
  const response = await axios.get(`todos/${state}`)
  return response.data
}

export const useTodosQuery = (state: State) =>
  useQuery({
    queryKey: ['todos', state],
    queryFn: () => fetchTodos(state),
    initialData: () => {
      const allTodos = queryClient.getQueryData<Todos>([
        'todos',
        'all',
      ])
      const filteredData =
        allTodos?.filter((todo) => todo.state === state) ?? []

      return filteredData.length > 0 ? filteredData : undefined
    },
  })
```


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/inside-react-query](https://tkdodo.eu/blog/inside-react-query)</small>
