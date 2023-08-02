---
share: true
tags:
 - gRPC/gRPC-web
 - gRPC/protobuf
---
# Генерируем заглушки TypeScript с помощью Protoc
Рассмотрим, как сгенерировать заглушки для языка TypeScript (TypeScript — язык для Angular). В отличие от Visual Studio и `dotnet`, которые делают это за нас, в данном случае нужно использовать утилиту Protoc самостоятельно, из командной строки. 
## Скачиваем правильную версию Protoc и Protobuf Well-known types
Последнюю версию Protoc можно скачать [отсюда](https://github.com/protocolbuffers/protobuf/releases). Выбирайте версию, подходящую для вашей операционной системы. Для Windows возьмем файл с `win64` в названии. В архиве будут папки **bin** с запускаемым файлом **protoc.exe**, и **include**, содержащую файлы `.proto` для [[ch-4-messages-well-known-types|Well-Known Types]]. Для удобства стоит добавить путь к **protoc.exe** в системную переменную Path.
Также нам понадобится файл `protobufs-*.zip` с исходниками Well Known Types. В этом `.zip`-файле найдём папку **google/protobufs** и скопируем к себе на диск, например в папку **C:\Protobufs**. %% возможно, не понадобится, тогда не забыть удалить этот абзац %%.
## Скачиваем плагин ts-protoc-gen
После того, как вы создали скелет приложения Angular под названием **CountryWiki.Angular** (проделайте это упражнение самостоятельно), установите плагин для Protoc, `ts-protoc-gen`:
```bash
npm install ts-protoc-gen
```
> [!note] От меня
> Я это сделал в папке проекта **CountryWiki.Angular**, не уверен, что это необходимо. Дальнейшие установки тоже буду выполнять внутри этой папки.
> Также нам потребуется, как мы это увидим дальше, плагин protoc-gen-js, скачаем его:
> ```bash
> npm install protoc-gen-js
> ```

## Скачиваем библиотеку gRPC-web от Improbable и библиотеку Protobufs от Google
```bash
npm install @improbable-eng/grpc-web
npm install @types/google-protobuf
npm install google-protobuf
```
## Выполняем команду Protoc
Для нашего проекта мы будем использовать те же самые `.proto`-файлы, которые сделали в [[ch-11-reworking-the-countryservice-grpc-service-for-browser-apps|главе 11]]. Добавим в наш Angular проект папку **protos/v1** и скопируем туда файлы **country.browser.proto** и **country.shared.proto**. Также нам нужно будет заранее создать папку под сгенерированные файлы, разместим её внутри проекта, в **src/app/generated**. Далее перейдем в папку **src** и выполним команду:
```bash
protoc --plugin=protoc-gen-js="{ABSOLUTE_PATH}/node_modules/.bin/protoc-gen-js.cmd" --plugin=protoc-gen-ts="{ABSOLUTE_PATH}/node_modules/.bin/protoc-gen-ts.cmd" --js_out="import_style=commonjs,binary:./app/generated" --ts_out="service=grpc-web:./app/generated" country.browser.proto country.shared.proto --proto_path="{ABSOLUTE_PATH}/src/protos/v1" --proto_path="C:/Protoc/include"
```
Вместо `{ABSOLUTE_PATH}` нужно подставить полный путь к папке с проектом. Обратите внимание, что нужно использовать обратный слэш `/` в качестве разделителя, даже если вы в Windows. Собственно, в Linux можно использовать относительные пути. Также, так как мы указали путь к файлам в `--proto_path`, в `country.browser.proto` нужно убрать путь к файлу `country.shared.proto`, то есть:
```protobuf
import "country.shared.proto";
```
Если всё прошло удачно, в папке **generated** будут следующие файлы:
![[Pasted image 20230715204444.png]]