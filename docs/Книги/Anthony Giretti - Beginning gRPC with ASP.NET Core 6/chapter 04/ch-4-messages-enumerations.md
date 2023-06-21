---
share: true
tags:
 - gRPC/protobuf/messages
---
# Перечисления (Enum)
Синтаксис для перечислений практически идентичен синтаксису в C\#. Однако, значения должны быть обязательно указаны и обязательно должны начинаться с нуля.
Также для перечислений существует специальная опция — `allow_alias = true`, при задании которой допускаются одинаковые значения для нескольких элементов enum-а. Пример:
```protobuf
message CountryReply {
	int32 CountryId = 1;
	Continent Continent = 2;
}
enum Continent {
	option allow_alias = true;
	Unknown = 0;
	NorthAmerica = 1;
	SouthAmerica = 2;
	Europe = 3;
	Africa = 4;
	Asia = 5;
	Australia = 6;
	Oceania = 6;
}
```
Сгенерированный код будет таким:
```csharp
public enum Continent {
	[pbr::OriginalName("Unknown")] Unknown = 0,
	[pbr::OriginalName("NorthAmerica")] NorthAmerica = 1,
	[pbr::OriginalName("SouthAmerica")] SouthAmerica = 2,
	[pbr::OriginalName("Europe")] Europe = 3,
	[pbr::OriginalName("Africa")] Africa = 4,
	[pbr::OriginalName("Asia")] Asia = 5,
	[pbr::OriginalName("Australia")] Australia = 6,
	[pbr::OriginalName("Oceania", PreferredAlias = false)] Oceania = 6
}
```
Также существует возможность резервировать значения перечисления для предотвращения ломающих изменений в будущем, об этом написано [здесь](https://protobuf.dev/programming-guides/proto3/#reserved-values)
