---
share: true
tags:
 - microservice/transaction
 - microservice/saga
---
# Поворотная транзакция
*Поворотная транзакция* — решающий момент в повествовании. Если поворотная транзакция фиксируется, повествование отрабатывает до конца. Поворотная транзакция может быть недоступной ни для компенсации, ни для повторения. Также это может быть последняя компенсируемая или первая повторяемая транзакция. 

Пример для повествования `Create Order`.
![[Pasted image 20210920202844.png]]