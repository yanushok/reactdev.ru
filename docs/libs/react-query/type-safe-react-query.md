---
description: Типобезопасный React Query
---

# Типобезопасный React Query

## Trust {#trust}
## Generics {#generics}
## About angle brackets {#about-angle-brackets}
## Lying angle brackets {#lying-angle-brackets}
### The golden rule of Generics {#the-golden-rule-of-generics}
## Trust again {#trust-again}
## zod {#zod}
### validation in the queryFn {#validation-in-the-queryfn}
### Tradeoffs {#tradeoffs}
## What about getQueryData {#what-about-getquerydata}
## End-to-end type-safety {#end-to-end-type-safety}







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


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/type-safe-react-query](https://tkdodo.eu/blog/type-safe-react-query)</small>
