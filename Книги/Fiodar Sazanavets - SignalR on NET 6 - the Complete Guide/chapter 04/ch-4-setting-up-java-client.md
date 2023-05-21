---
share: true
tags:
 - NET/SignalR
 - Java
---
# Настраиваем Java клиент
## Настраиваем проект
Прежде всего, нам нужно создать проект. Этот процесс выглядит по разному в зависимости от того, используется ли система сборки Maven или Gradle.
### Генерируем шаблон проекта с помощью Gradle
Выполним в желаемой папке
```bash
gradle init
```
Далее, вам будет предложено ввести значения нескольких параметров вашего проекта. Выбирайте, что хотите, однако убедитесь, что выбран язык Java, а также тип проекта application.
Пакет SignalR может работать и с другими языками, такими как Kotlin, однако в этом разделе мы приведём примеры на языке Java.
### Генерируем шаблон проекта с помощью Maven
При использовании Maven проект необходимо создавать из так называемого *архетипа*. Это аналог шаблона проекта в .NET. Одним из простейших архетипов является `maven-archetype-quickstart`. Для создания проекта из этого архетипа выполним следующую команду:
```bash
mvn archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4
```

---
Вне зависимости от того, использовали ли мы Maven или Gradle, в результате мы должны получить структуру каталогов с файлом **App.java** с методом `main` внутри.
## Добавляем зависимость
Прежде всего нужно посетить [эту страницу](https://search.maven.org/artifact/com.microsoft.signalr/signalr), чтобы узнать последнюю версию пакета SignalR. Далее, кликнув на номере этой версии мы попадём на страницу с примерами синтаксиса добавления зависимости для различных менеджеров сборки.
## Добавляем код клиента SignalR в приложение
Открываем файл **App.java** и пишем там следующее:
```java
// Нужные импорты
import com.microsoft.signalr.HubConnection;
import com.microsoft.signalr.HubConnectionBuilder;
import java.util.Scanner;

public static void main(String[] args) throws Exception {
	// Читаем URL из консоли
	System.out.println("Please specify the URL of SignalR Hub");
	Scanner reader = new Scanner(System.in);
	String input = reader.nextLine();
	// Конфигурируем подключение к хабу и обработчик клиентского метода
	HubConnection hubConnection = HubConnectionBuilder.create(input).build();
	hubConnection.on("ReceiveMessage", (message) -> {
		System.out.println(message);
	}, String.class);
	// Подключаемся к хабу и отправляем все сообщения из консоли в BroadcastMessage
	// выход по тексту exit
	hubConnection.start().blockingAwait();
	while (!input.equals("exit")){
		input = reader.nextLine();
		hubConnection.send("BroadcastMessage", input);
	}
	hubConnection.stop();
}
```
## Запускаем клиент
Если вы используете Gradle, приложение можно запустить при помощи команды `gradle run`. В других случаях это может потребовать несколько большего количества действий, обратитесь к документации вашей системы сборки.
URL хаба получаем [[ch-4-setting-up-net-client|точно также]], как и в случае с NET клиентом.