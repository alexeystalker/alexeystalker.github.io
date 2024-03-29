---
share: true
tags:
 - event-sourcing
---
# Улучшение производительности с помощью снимков
В случае, когда агрегат достаточно долговечен, и его состояние часто меняется, со временем у него накапливается довольно много событий, что приводит к замедлению его загрузки и свёртки.
Эту проблему решают периодическим сохранением снимков состояния агрегата.
![[Pasted image 20210929203331.png]]
Приложение восстанавливает состояние агрегата, загружая его последний снимок и только те события, которые произошли с момента его создания.
Если у агрегата простая, легко сериализуемая структура, в качестве снимка может выступать его сериализация в JSON. Снимки более сложных агрегатов создаются с помощью [[memento-pattern|шаблона "Хранитель"]]