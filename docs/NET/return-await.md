---
share: true
tags: [NET/async]
---
# Избегаем `return await`
Рассмотрим код
```csharp
public async Task CallAsync()
{
	var client = new Client();
	return await client.GetAsync();
}
```
Код семантически корректен, однако нет необходимости делать `return await` — это приведет к ненужному оверхеду при компиляции.

Если в методе только один вызов асинхронного метода, и нет необходимости в дополнительных действиях после вызова метода — лучше убрать `async/await` и ограничиться следующим:
```csharp
public Task CallAsync()
{
	var client = new Client();
	return client.GetAsync()
}
```
Однако нужно помнить, что если нам нужно обернуть вызов в `try/catch` или `using` — `return await` придется использовать.
