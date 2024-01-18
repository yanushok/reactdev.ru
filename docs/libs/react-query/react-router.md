---
description: Взаимодействие React Query и React Router при работе вместе
---

# React Query и React Router

## A router that fetches data {#a-router-that-fetches-data}
### It's not a cache {#its-not-a-cache}
### Fetching early {#fetching-early}
### Fetching too often {#fetching-too-often}
## Querifying the example {#querifying-the-example}
### The loader needs access to the QueryClient. {#the-loader-needs-access-to-the-queryclient}
### getQueryData ?? fetchQuery {#getquerydata--fetchquery}
### A TypeScript tip {#a-typescript-tip}
## Invalidating in actions {#invalidating-in-actions}
### await is the lever {#await-is-the-lever}
## Summary {#summary}







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


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/react-query-meets-react-router](https://tkdodo.eu/blog/react-query-meets-react-router)</small>
