---
description: Работа с формами в React Query
---

# Работа с формами в React Query

## Server State vs. Client State {#server-state-vs-client-state}
## The simple approach {#the-simple-approach}
### Data might be undefined {#data-might-be-undefined}
### No background updates {#no-background-updates}
## Keeping background updates on {#keeping-background-updates-on}
### You need controlled fields {#you-need-controlled-fields}
### Deriving state might be difficult {#deriving-state-might-be-difficult}
## Tips and Tricks {#tips-and-tricks}
### Double submit prevention {#double-submit-prevention}
### Invalidate and reset after mutation {#invalidate-and-reset-after-mutation}







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


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/react-query-and-forms](https://tkdodo.eu/blog/react-query-and-forms)</small>
