---
description: Возможно, вам не нужен React Query
---

# Возможно, вам не нужен React Query


## Мое мнение {#my-take}
### Что же изменилось? {#so-what-changed}
### Возможно, он вам не нужен {#you-might-not-need-it}
### Интеграция {#integration}
### Гибридный подход {#hybrid-approach}
### Это не "убийца" {#its-not-a-killer}







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


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/you-might-not-need-react-query](https://tkdodo.eu/blog/you-might-not-need-react-query)</small>
