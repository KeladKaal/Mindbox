# Тестовое задание Mindbox

В задаче два основных пункта: отказоустойчивость и минимальное потребление ресурсов. Рассмотрим отдельно каждый из них:

**Отказоустойчивость**

Кластер мультизональный, с несколькими нодами. Предположу, что ноды равномерно разнесены по зонам. Следовательно, логично распределять поды между ними. Для этого в моём случае использовано `topologySpreadConstraints`

Отказоустойчивость во многом обеспечивает сам Kubernetes, за счёт проверки работы подов и их перезапуска в случае ошибок. Здесь была добавлена отсрочка в 10 секунд в `livenessProbe`, так как по условию необходимо 5-10 секунд для начала нормальной работы.


**Минимальное потребление ресурсов**

Чтобы не занимать лишних ресурсов, необходимо автоматически отслеживать их использование. В Kubernetes есть два основных инструмента для автомасштабирования — HPA и VPA. Поскольку здесь можно безболезненно изменять количество подов, есть достаточно прозрачные критерии по использованию ресурсов — очевидно, что необходимо использовать HPA. 

Из условий задания, здесь есть ограничения по количеству реплик:   `minReplicas: 1`
 и `maxReplicas: 4`.  Поскольку по памяти всегда одинаковое потребление, а основная загрузка приходится на CPU — HPA будет масштабировать количество подов, исходя из их загрузки. Поскольку дневная и ночная нагрузка волнообразная, то с ней он также должен справиться. В продакшене после тестирования, возможно, потребовалось бы прописать политики `Scale Up` и `Scale Down` во избежание поддержки слишком большого количества подов лишнее время.

**Итоги:**

Для обеспечения отказоустойчивости поды равномерно распределяются по зонам доступности. Для экономии ресурсов используется `HorizontalPodAutoscaler`, который отслеживает нагрузку на CPU и исходя из неё меняет количество подов.