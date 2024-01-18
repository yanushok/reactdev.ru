---
description: Обработка ошибок в React Query
---

# Обработка ошибок в React Query

## Предпосылки {#prerequisites}
## Стандартный пример {#the-standard-example}
## Границы ошибок {#error-boundaries}
## Отображение уведомлений об ошибках {#showing-error-notifications}
### Коллбэк onError {#the-onerror-callback}
### Глобальный коллбэк {#the-global-callbacks}
## Собираем все вместе {#putting-it-all-together}







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


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/react-query-error-handling](https://tkdodo.eu/blog/react-query-error-handling)</small>
