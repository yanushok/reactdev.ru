---
description: Заполнение кэша запросов
---

# Заполнение кэша запросов

## Fetch waterfalls {#fetch-waterfalls}
### Suspense {#suspense}
### Suspense waterfalls {#suspense-waterfalls}
## Prefetching {#prefetching}
### The use RFC {#the-use-rfc}
## Seeding details from lists {#seeding-details-from-lists}
### Pull approach {#pull-approach}
### Push approach {#push-approach}







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


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/seeding-the-query-cache](https://tkdodo.eu/blog/seeding-the-query-cache)</small>
