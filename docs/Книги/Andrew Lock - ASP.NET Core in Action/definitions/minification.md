---
share: true
tags:
 - definition
---
# Минификация
*Минификация (minification)* — процесс обработки кода для уменьшения его размера без изменения поведения. Включает в себя удаление пробелов и комментариев, задание переменным более коротких имен и полное удаление неиспользуемых разделов кода.
## Пример минификации
Пусть есть такой код:
```js
function myFunc() {
	// на самом деле эта функция ничего не делает,
	// она здесь просто для того, чтобы продемонстрировать минификацию.
	function innerFunctionToAddTwoNumbers(theFirstNumber, theSecondNumber) {
		// функция внутри функции myFunc
		return theFirstNumber + theSecondNumber;
	}
	var shouldAddNumbers = true;
	var totalOfAllTheNumbers = 0;
	
	if (shouldAddNumbers == true) {
		for (var index = 0; i < 10; i++) {
			totalOfAllTheNumbers =
				innerFunctionToAddTwoNumbers(totalOfAllTheNumbers, index);
		}
	}
	return totalOfAllNumbers;
}
```
Подвергнем этот код минификации:
```js
function myFunc(){function r(n,t){return n+t}var n=0,t;if(1)for(t=0;i<10;i++)n=r(n,t);return n}
```
Как видим, код уменьшился с 588 до 95 байт.