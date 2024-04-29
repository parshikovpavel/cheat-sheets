*Package* `container/heap` предоставляет *heap operation*'s для любого *type*, который *implement* `heap.Interface`. *Heap* — это *tree*, свойство которого состоит в том, что каждая *node* является *minimum-value node* в своем *subtree* (для *min-heap*, [подробнее здесь](https://github.com/parshikovpavel/data-structure/tree/main/heap)).

Минимальным элементом дерева является корень с индексом 0.

*Heap* — это распространенный способ реализации *priority queue*. Чтобы построить *priority queue*, *implement*'ируйте *Heap interface* с (отрицательным) *priority* в качестве порядка для `Less()` *method*, чтобы `Push()` добавлял, а `Pop()` удалял *highest-priority item*.  Ниже приведен пример. 

**Пример (`IntHeap`):**

Этот пример делает:

-  `Init()` для `IntHeap` из нескольких *int*'ов
-  `Push()`
- поолучает минимум
- `Pop()` элементы в порядке приоритета.

```go
// Этот пример демонстрирует integer heap, построенную на heap interface.
package main

import (
	"container/heap"
	"fmt"
)

// IntHeap – min-heap из int'ов.
type IntHeap []int

func (h IntHeap) Len() int           { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *IntHeap) Push(x any) {
	*h = append(*h, x.(int)) // h = [1, 2, 5, 3]
}

func (h *IntHeap) Pop() any {
	old := *h // h = [2, 3, 5, 1]
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1] // h = [2, 3, 5]
	return x
}

func main() {
	h := &IntHeap{2, 1, 5}  // h = [2, 1, 5]
	heap.Init(h)            // h = [1, 2, 5]
	heap.Push(h, 3)         // h = [1, 2, 5, 3]
	fmt.Printf("minimum: %d\n", (*h)[0]) // minimum: 1
	for h.Len() > 0 {
		fmt.Printf("%d ", heap.Pop(h)) // 1, 2, 3, 5
	}
}

```

**Пример (`PriorityQueue`)**

В этом примере:

- создается `PriorityQueue` с 3 элементами
- *push* элемент
- *update* элемент
- *pop* элементы в порядке приоритета.

```go
// Пример демонстрирует priority queue, построенная с использованием heap interface.
package main

import (
	"container/heap"
	"fmt"
)

// Item - элемент в priority queue
type Item struct {
	value    string
	priority int
	// index обновляется вручную в method'ах и нужен для выпонения Fix()
	index int
}

// PriorityQueue implement'ит heap.Interface и содержит Items.
type PriorityQueue []*Item

func (pq PriorityQueue) Len() int { return len(pq) }

func (pq PriorityQueue) Less(i, j int) bool {
	// Мы хотим, чтобы Pop возвращал Item с самым высоким priority, поэтому ставим знак больше (>)
	return pq[i].priority > pq[j].priority
}

func (pq PriorityQueue) Swap(i, j int) {
	pq[i], pq[j] = pq[j], pq[i]
	pq[i].index = i
	pq[j].index = j
}

func (pq *PriorityQueue) Push(x any) {
	n := len(*pq)
	item := x.(*Item)
	item.index = n
	*pq = append(*pq, item)
}

func (pq *PriorityQueue) Pop() any {
	old := *pq
	n := len(old)
	item := old[n-1]
	old[n-1] = nil  // избежать memory leak, т.к. при уменьшении slice - pointer хранится в underlying array
	item.index = -1 // для безопасности
	*pq = old[0 : n-1]
	return item
}

// update модифициует priority и value для Item. index извлекает из Item
func (pq *PriorityQueue) update(item *Item, priority int) {
	item.priority = priority
	heap.Fix(pq, item.index)
}

func main() {
	items := map[string]int{
		"banana": 3, "apple": 2, "pear": 4,
	}

	// Создает priority queue, добавляет item'ы,
	// устанавливает invariant'ы для priority queue (heap)
	pq := make(PriorityQueue, len(items))
	i := 0
	for value, priority := range items {
		pq[i] = &Item{
			value:    value,
			priority: priority,
			index:    i,
		}
		i++
	}
	// [3, 2, 4]
	heap.Init(&pq) // [4, 2, 3]
	// Вставить новый item
	item := &Item{
		value:    "orange",
		priority: 1,
	}
	heap.Push(&pq, item) // [4, 2, 3, 1]
	// update его priority
	pq.update(item, 5) // [5, 4, 3, 2]

	// Pop items в порядке убывания priority.
	for pq.Len() > 0 {
		item := heap.Pop(&pq).(*Item)
		// 05:orange 04:pear 03:banana 02:apple
		fmt.Printf("%.2d:%s ", item.priority, item.value)
	}
}
```





# Function

Во всех *function*'s (или некоторые?) должен передаваться *pointer* на *slice*, т.к эти *function*'s модифицируют *slice's length*, а не только содержимое

## `Fix()`

```go
func Fix(h Interface, i int)
```

`Fix()` восстанавливает *heap invariant*'ы после того, как элемент с *index*'ом `i` изменил свое *value*. Изменение *value* элемента с *index*'ом `i` и последующий вызов `Fix()` эквивалентно, но имеет меньшую *time complexity*, чем вызов `Remove(h, i)` с последующим `Push(h, new_value)`. 

*Time complexity* равна `O(log n)`, где `n = h.Len()`.

## `Init()`

```go
func Init(h Interface)
```

`Init()` устанавливает *heap invariant*'ы, необходимые для других *function*'s в этом пакете. `Init()` – *idempotent*'на по отношению к *heap invariant*'ам и может вызываться всякий раз, когда *heap invariant*'ы были нарушены. 

*Time complexity* равна `O(n)`, где `n = h.Len()`.

## `Pop()`

```go
func Pop(h Interface) any
```

`Pop()` удаляет и возвращает *minimum element* (в соответствии с `Less()`) из *heap*. *Complexity* равна `O(log n)` где `n = h.Len()`. `Pop()` эквивалентен `Remove(h, 0)`.

Реализация `Pop()` в библиотеке [в соответствии с алгоритмом *delete* для *heap*](https://github.com/parshikovpavel/data-structure/tree/main/heap#delete): 

- делает *swap* для *root* и *last element* на *last level*;
- уменьшает размер *heap* на 1; 
- выполняет для *heap* операцию *sift-down*, 
- вызывает `Interface.Pop()` и ожидает, что в этой функции:
  - *last element* (который ранее был *root*) – будет извлечен из *heap* и возвращен как результат операции
  - уменьшен размер *heap* на 1

## `Push()`

```go
func Push(h Interface, x any)
```

`Push()` помещает элемент `x` в *heap*. *Complexity* равна `O(log n)`, где `n = h.Len()`.

Реализация `Push()` в библиотеке [в соответствии с алгоритмом *insert* для *heap*](https://github.com/parshikovpavel/data-structure/tree/main/heap#insert-push): 

- вызывает `Interface.Push()` и ожидает, что в этой функции:
  - Элемент `x` будет *append* в качестве *last element*'а
  - При *append* размер *heap* увеличивается на 1
- выполняет для *heap* операцию *sift-up*, 

## `Remove()`

```go
func Remove(h Interface, i int) any
```

`Remove()` удаляет и возвращает элемент с *index*'ом `i` из *heap*. 

*Complexity* равна `O(log n)`, где `n = h.Len()`.

Для этого:

- делает *swap* для элемента с *index*'ом `i`  и *last element* на *last level*
- уменьшает размер *heap* на 1
- выполняет действия аналогичные `Fix()` с *index*'ом `i`
- вызывает `Interface.Pop()` и ожидает, что в этой функции:
  - *last element* (который ранее был *root*) – будет извлечен из *heap* и возвращен как результат операции
  - уменьшен размер *heap* на 1

# Type

## `type Interface`

```go
type Interface interface {
	sort.Interface
  /*
  Push() должен:
  - append элемент x в качестве last element'а
  - при append размер (len) heap увеличивается на 1
  */
	Push(x any)
  
  /*
  Pop() должен:
  - извлечь из heap - last element (Len()-1), который ранее был root, и вернуть его как результат операции
  - уменьшить размер heap на 1
  */
	Pop() any   
}
```

`Interface` *type* описывает требования к *type*, использующему *function*'s в этом *package*. Любой *type*, который *implement*'ит `Interface`, может использоваться как *min-heap* со следующими *invariant*'ами (они устанваливаются после вызова `Init()`):

```
!h.Less(j, i) для 0 <= i < h.Len(), где 2*i+1 <= j <= 2*i+2 и j < h.Len()
```

`Interface.Push()` и `Interface.Pop()` предназначены для вызова внутри *function*'s из этого *package*, их не нужно вызывать из пользовательского кода. Чтобы *push* и *pop* элементы из *heap*, используйте `heap.Push()` и `heap.Pop()`.