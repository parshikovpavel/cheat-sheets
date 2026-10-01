# Go 1.27

- Стали возможны [Generic Methods](types/generic.md#generic-methods):

  ```go
  type List[T any] []T
  
  func (s List[T]) Map[U any](f func(T) U) List[U] {
      out := make(List[U], len(s))
      
      for i, v := range s {
          out[i] = f(v)
      }
      
      return out
  }
  
  func main() {
      nums := List[int]{1, 2, 3}
      strs := nums.Map(strconv.Itoa)
      fmt.Printf("%q\n", strs) // ["1" "2" "3"]
  }
  ```

- *Promoted field* в  Struct-литералах доступны без вложенных обёрток ([ссылка](types/struct.md#promoted-field-и-method))

  ```go
  type A struct {
    B  // embedded field, field name is B
  }
  
  type B struct {
    field int
  }
  
  a := A{field: 1}
  ```



# Go 1.26

- Поддержка *expression* в `new()` ([ссылка](package/builtin.md#new)) 

  ```go
  // раньше вводили свои кастомные функции типа
  enabled := ToPtr(true)
  
  // теперь через new
  enabled := new(true)
  ```

  



# Go 1.25

- Добавление метода `sync.WaitGroup.Go()`- для запуска *goroutine* в группе без ручных `Add()` и `Done()` ([ссылка](packages/sync.md#WaitGroup.Go()))
