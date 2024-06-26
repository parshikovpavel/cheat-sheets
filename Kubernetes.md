https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

https://blog.newrelic.com/engineering/kubernetes-request-and-limits/

Kubernetes - это система для автоматических контейнеризированных приложений. Он управляет *node*'s в вашем кластере, а вы определяете требования к ресурсам для своих приложений. 

### Request and limit для CPU и Memory

Каждая *node* в *Kubernetes cluster* имеет выделенный объем *memory* (RAM) и *computational power* (CPU), которые можно использовать для запуска *container*'s.

Kubernetes выполняет логическую группировку одного или нескольких *container*'s в *pod*'s. Pod's, в свою очередь, можно деплоить и управлять ими поверх *node*'s. Когда вы создаете *pod*, вы обычно указываете *storage* и *networking*, которые *container*'s  совместно используют в этом *pod*. *Kubernetes scheduler* затем будет искать *node*'s, у которых есть ресурсы, требуемые (*required*) для запуска *pod*'а.

Чтобы помочь scheduler'у, мы можем указать нижний и верхний предел RAM и CPU для каждого *container*'а, используя:

- *container request* – минимальное количество RAM и CPU, требуемый для этого *container*'а. Kubernetes объединит все *container request*'s в общий *pod request*. *Scheduler* будет использовать этот общий *pod request*, чтобы гарантировать, что pod может быть задеплоем на *node* с достаточным количеством ресурсов.
- *container limit* – максимальное количество RAM или CPU, которые может потреблять *container*. Kubernetes транслирует  *limit* в сервис контейнеризации (Docker), которая применяет *limit*. Если контейнер превышает *memory limit*,  ядро системы завершает процесс, который попытался выделить память, с ошибкой out of memory (OOM).. *CPU limit* менее строг и, как правило, может превышаться в течение длительных периодов времени (тротлинг?)

- 

Посмотрим, как используются запросы и лимиты.

### Измерение CPU

*Limit* и *request* для CPU измеряются в единицах `cpu`. Один `cpu` эквивалентен 1 vCPU (1 виртуальный CPU или ядро).

Допускаются дробные значения. Например `cpu=0.5`гарантированно будет вдвое меньше `cpu = 1` (1 CPU). Выражение `0.1` эквивалентно выражению `100m`, которое можно прочитать как «сто millicpu» или «сто millicores». Значения меньше  `1m` не допускаются. 

CPU всегда запрашивается как абсолютное количество, а не как относительное количество; `0,1` - это такое же количество CPU на одноядерном, двухъядерном или 48-ядерном компьютере.

### Измерение memory

*Limit* и *request* для *memory* измеряются в байтах. Вы можете выразить память как простое целое число или как число с фиксированной точкой, используя один из этих суффиксов: `E`, `P`, `T`, `G`, `M`, `K`. При этом `1 Kilobyte (K) = 1,000 bytes`, `1 Megabyte (M) = 1,000,000 bytes`, `1 gigabytes (G) = 1,000,000,000 bytes`. 

Можно также использовать сокращения, которые получены через степень двойки: `Ei`, `Pi`, `Ti`, `Gi`, `Mi`, `Ki`. Т.е. `1 Ki= 1,024 bytes` . Для memory дробные значения не допускаются, наименьшая величина – 1 byte.





### ingress (api-gateway)

TODO!!!