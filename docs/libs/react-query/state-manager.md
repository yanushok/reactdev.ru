---
description: React Query как стейт менеджер
---

# React Query как стейт менеджер
## Асинхронный менеджер состояний {#an-async-state-manager}
## Инструмент для синхронизации данных {#a-data-synchronization-tool}
### До React Query {#before-react-query}
### Устаревшие данные при повторной проверке {#stale-while-revalidate}
### Умный рефетч {#smart-refetches}
### Позволяем React Query творить волшебство {#letting-react-query-do-its-magic}
### Настроить staleTime {#customize-staletime}
#### Бонус: использование setQueryDefaults {#bonus-using-setquerydefaults}
## Замечание о разделении забот {#a-note-on-separation-of-concerns}
## Заключение {#takeaways}







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


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/react-query-as-a-state-manager](https://tkdodo.eu/blog/react-query-as-a-state-manager)</small>
