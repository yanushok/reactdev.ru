---
description: Описание API опций запроса
---

# API опций запроса

## Лучшая абстракция {#a-better-abstraction}
## TypeScript {#typescript}
## queryOptions {#queryoptions}
### DataTag {#datatag}
## Фабрики запросов {#query-factories}







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


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/the-query-options-api](https://tkdodo.eu/blog/the-query-options-api)</small>
