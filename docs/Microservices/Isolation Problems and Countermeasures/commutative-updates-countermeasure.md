---
share: true
tags:
 - microservice/transaction
 - microservice/isolation/countermeasure
---
# Контрмера “коммутативные обновления”
Операции обновления необходимо по возможности проектировать коммутативными. В этом случае не будет проблем с выполнением [[saga-compensating|компенсирующих]] транзакций даже, если между компенсируемой и компенсирующей транзакциями были другие.