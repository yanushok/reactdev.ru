---
description: Взаимодействие React Query и React Context при работе вместе
---

# React Query и React Context


## Быть самодостаточным {#being-self-contained}
### Неявная зависимость {#an-implicit-dependency}
### Сделайте это явным {#make-it-explicit}
## React Context {#react-context}
### Приятный TypeScript {#pleasing-typescript}
### Синхронизация состояний {#state-syncing}
### Водопады запросов {#request-waterfalls}
### Suspense {#suspense}







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


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/react-query-and-react-context](https://tkdodo.eu/blog/react-query-and-react-context)</small>
