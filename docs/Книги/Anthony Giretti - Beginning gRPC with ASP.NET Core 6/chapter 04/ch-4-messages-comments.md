---
share: true
tags:
 - gRPC/protobuf/messages
---
# Комментарии
В файлы Protobuf можно добавлять комментарии. Возможный синтаксис:
```protobuf
/* Многострочный комментарий */
// Однострочный комментарий
```
При этом, если комментарии будут *окружать* код, который должен попасть в генерируемый код, то комментарии тоже будут перенесены в XmlComments (те, что с тремя слэшами `///` и XML-разметкой). Например, пусть у нас есть `.proto` файл с комментариями
```protobuf
/*
Author: Anthony Giretti
Example for a book
*/
syntax = "proto3";
package gRPCDemo.v1;
option csharp_namespace = "Sample.gRPC";

//The Error message entity
message Error {
	string SearchContent = 1; // The initial search keyword
	string Description = 2; // The error description
}
```
Тогда в сгенерированном коде мы получим
```csharp
/// <summary>
/// The Error message entity
/// </summary>
public sealed partial class Error : pb::IMessage<Error>
{
	/// <summary>Field number for the "SearchContent" field.</summary>
	public const int SearchContentFieldNumber = 1;
	private string searchContent_ = "";
	/// <summary>
	/// The initial search keyword
	/// </summary>
	[global::System.Diagnostics.DebuggerNonUserCodeAttribute]
	public string SearchContent 
	{
		get { return searchContent_; }
		set {
			searchContent_ = pb::ProtoPreconditions.CheckNotNull(value, "value");
		}
	}
	
	/// <summary>Field number for the "Description" field.</summary>
	public const int DescriptionFieldNumber = 2;
	private string description_ = "";
	/// <summary>
	/// The error description
	/// </summary>
	[global::System.Diagnostics.DebuggerNonUserCodeAttribute]
	public string Description
	{
		get { return description_; }
		set {
			description_ = pb::ProtoPreconditions.CheckNotNull(value, "value");
		}
	}
}
```
Видно, что наши комментарии попали в XmlComments.