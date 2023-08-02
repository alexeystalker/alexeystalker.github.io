---
share: true
tags:
 - gRPC/gRPC-web
---
# Доступ к данным с помощью клиента gRPC-web от Improbable
Написать клиент для gRPC-web легко. Improbable предлагает общую фукцию для управления как [[grpc-unary-call|унарными вызовами]], так и [[grpc-server-streaming-call|серверными потоками]], метод `grpc.invoke`, который вызывается так:
```ts
grpc.invoke(methodDescriptor: MethodDescriptor, props: InvokeRpcOptions)
```
`MethodDescriptor` — это метод, который вы хотите вызвать. Методы определены в сгенерированном файле `{ProtobuFname}_pb_services.d.ts` как свойства класса, обозначающего сервис gRPC-web:
![[Pasted image 20230716215812.png]]
Вот сервис `CountryServiceBrowser` и метод `CountryServiceBrowserCreate`.  Если мы захотим вызвать метод `Create`, то MethodDescriptor-ом будет `CountryServiceBrowser.Create`.
Второй параметр, `InvokeRpcOptions`, имеет следующие свойства:
- `host` — URI сервера, например `https://localhost:5001`;
- `request` — сообщение, отправляемое на сервер, например, `new Empty()`;
- `metadata` — [[ch-3-trailers|трейлеры]], которые будут отправлены на сервер, например, `new grpc.Metadata({"TrailerKey": "TrailerValue"})`;
- `onHeaders` — коллбэк, обрабатывающий заголовки, пришедшие с сервера, например, `(headers: grpc.Metadata) => { const headersValue = headers.get("HeaderKey"); }`;
- `onMessage` — коллбэк, обрабатывающий сообщения, пришедшие с сервера, например, `(countryReply: CountryReply) => { const country => countryReply.toObject(); }`. Обратите внимание, что каждое сообщение должно быть десериализовано методом `toObject()`. Коллбэк вызывается один раз в случае унарного вызова или на каждое сообщение в случае серверного потока;
- `onEnd` — коллбэк, вызываемый в конце вызова и позволяющий обработать данные, полученные от сервера в завершение вызова: gRPC-статус, трейлеры, и строковое сообщение; например, `(code: grpc.Code, msg: string | undefined, trailers: grpc.Metadata, endMessage: String) => { if (code !== grpc.Code.Ok) { ... } else { ... } }`;
- `transport` — опциональное свойство, позволяет отправлять куки на сервер вместе с cross-origin запросами; например, `grpc.CrossBrowserHttpTransport({ withCredentials: true });`[^1]
- `debug` — опциональное свойство, позволяющее выводить отладочную информацию в консоль.

Обратите внимание, что метод `grpc.invoke()` возвращает объект `Request`, содержащий метод `close()`. Если вызвать этот метод так, как показано, то запрос будет *отменён*, а соединение с сервером *закрыто*:
```ts
const request = grpc.invoke(..., ...);
request.close();
```

Теперь рассмотрим реализацию `countryService`, которую мы поместим в файл **src/app/services/countryService.ts**:
```ts
import { grpc } from "@improbable-eng/grpc-web";
import { Empty } from "google-protobuf/google/protobuf/empty_pb";
import { CountryServiceBrowser } from "../generated/country.browser_pb_service";
import { CountriesCreationRequest, CountryCreationReply, CountryIdRequest, CountryReply, CountryUpdateRequest } from "../generated/country.shared_pb";
import { CountryCreationModelMapper } from "../mappers/countryCreationModelMapper";
import { CountryReplyMapper } from "../mappers/countryReplyMapper";
import { CountryCreationModel } from "../models/countryCreationModel";
import { CountryModel } from "../models/countryModel";
import { environment } from "../../environments/environment";
import { Injectable } from "@angular/core";
import { CountryUpdateModel } from "../models/countryUpdateModel";
import { UploadResultModel } from "../models/uploadResultModel";

@Injectable()
export class CountryService {
  public GetAll(countries: CountryModel[]): void {
    grpc.invoke(CountryServiceBrowser.GetAll,
      {
        request: new Empty(),
        host: environment.host,
        onMessage: (countryReply: CountryReply) => {
          let country = new CountryModel();
          CountryReplyMapper.Map(country, countryReply.toObject())
          countries.push(country);
        },
        onEnd: (code: grpc.Code, msg: string | undefined, trailers: grpc.Metadata) => this.onEnd(code,
          msg,
          trailers,
          "All countries have been downloaded")
      });
  }

  public Create(countriesToCreate: CountryCreationModel[], uploadResult: UploadResultModel, callback: Function): void {
    let countriesCreationRequest = new CountriesCreationRequest();
    CountryCreationModelMapper.Maps(countriesCreationRequest, countriesToCreate);
    grpc.invoke(CountryServiceBrowser.Create,
      {
        request: countriesCreationRequest,
        host: environment.host,
        onEnd: (code: grpc.Code, msg: string | undefined, trailers: grpc.Metadata) => {
          uploadResult.isProcessing = false;
          callback();
          this.onEnd(code, msg, trailers, "All countries have been created")
        }
      });
  }

  public Delete(id: number): void {
    let request = new CountryIdRequest();
    request.setId(id);
    grpc.invoke(CountryServiceBrowser.Delete,
      {
        request: request,
        host: environment.host,
        onEnd: (code: grpc.Code, msg: string | undefined, trailers: grpc.Metadata) => this.onEnd(code, msg, trailers, 'Country with Id ${id} has been deleted')
      });
  }

  public Get(id: number, country: CountryModel): void {
    let request = new CountryIdRequest();
    request.setId(id);
    grpc.invoke(CountryServiceBrowser.Get,
      {
        request: request,
        host: environment.host,
        onMessage: (countryReply: CountryReply) => {
          CountryReplyMapper.Map(country, countryReply.toObject());
        },
        onEnd: (code: grpc.Code, msg: string | undefined, trailers: grpc.Metadata) => this.onEnd(code,
          msg,
          trailers,
          'Country with Id ${id} was successfully found')
      });

  }

  public Update(countryUpdateModel: CountryUpdateModel): void {
    let request = new CountryUpdateRequest();
    request.setId(countryUpdateModel.id);
    request.setDescription(countryUpdateModel.description);
    grpc.invoke(CountryServiceBrowser.Update,
      {
        request: request,
        host: environment.host,
        onEnd: (code: grpc.Code, msg: string | undefined, trailers: grpc.Metadata) => this.onEnd(code,
          msg,
          trailers,
          'Country with Is ${countryUpdateModel.id} was successfully updated')
      });
  }

  private onEnd(code: grpc.Code, msg: string | undefined, trailers: grpc.Metadata, endMessage: String): void {
    if (code == grpc.Code.OK) {
      console.log(endMessage);
    } else {
      console.log('Hit an error status: ${grpc.Code[code]}');
      if (msg) {
        console.log('message: ${msg}');
      }
      trailers.forEach(trailer => {
        console.log('with the trailer ${trailer}: ${trailers.get(trailer)}');
      });
    }
  }
}
```
Здесь:
- общий метод `onEnd`, в котором выполняется логирование;
- `GetAll()` получает массив `CountryModel`, в который складывает полученные из потока сообщений `CountryReply` в методе, указанном в `onMessage`. Мы преобразуем каждое сообщение в объект модели `CountryModel` в статическом методе `CountryReplyMapper.Map`, который мы рассмотрим чуть позже;
- `Create()` получает массив объектов `CountryCreationModel`, которые преобразовывает в `CountryCreationRequest`, а также переменную `UploadResultModel`, которую мы заполняем в методе `onEnd`, отмечая, что загрузка выполнена. Последний параметр — `callback`, это функция, которую вызывает метод `onEnd` после окончания загрузки. Преобразование выполняется в статическом методе `CountryCreationModelMapper.Maps`;
- `Delete` получает идентификатор записи, заполняет объект `CountryIdRequest` и отправляет запрос на удаление;
- `Get` получает идентификатор записи, которую требуется загрузить, и переменную, в которую нужно сложить результат. Преобразование выполняется в статическом методе ` CountryReplyMapper.Map`;
- `Update()` получает объект `CountryUpdateModel`, заполняет объект `CountryUpdateRequest` и отправляет запрос.

Теперь приведём использованные модели, файлы которых разместим в папке **src/app/models**.
```ts
import { ActionResultModel } from "./actionResultModel";
import { CountryCreationModel } from "./countryCreationModel";  

export class UploadResultModel extends ActionResultModel {
  payload!: CountryCreationModel[];
  isProcessing!: boolean;
}
```
```ts
export class ActionResultModel {
  success!: boolean;
  errorMessage!: String;
}
```
```ts
export class CountryModel {
  id!: number;
  name!: String;
  description!: String;
  capitalCity!: String;
  anthem!: String;
  languages!: String[];
  flagUri!: String;
}
```
```ts
export class CountryUpdateModel {
  id!: number;
  description!: string;
}
```
Для `CountryCreationModel` нам потребуется ещё одна библиотека, `ts-json-object`. Эта библиотека позволяет добавлять аннотации к объявлению свойств класса с данными, например, пометить свойство как обязательное. При этом, если свойство не будет заполнено, возникнет исключение.
Чтобы всё работало, ваш класс должен быть унаследован от класса `JSONObject`, который также предоставляет конструктор, в который можно передать объект анонимного класса, из которого будут заполнены свойства.
Вот класс `countryCreationModel` с аннотированными свойствами.
```ts
import { JSONObject } from "ts-json-object"  

export class CountryCreationModel extends JSONObject {
  @JSONObject.required
  name!: string;  

  @JSONObject.required
  description!: string;  

  @JSONObject.required
  capitalCity!: string;  

  @JSONObject.required
  anthem!: string;  

  @JSONObject.required
  flagUri!: string;  

  @JSONObject.required
  languages!: number[];
}
```
В завершение раздела приведём код мапперов `CountryCreationMapper` и `CountryReplyMapper`.
```ts
import { CountriesCreationRequest, CountryCreationRequest } from "../generated/country.shared_pb";
import { CountryCreationModel } from "../models/countryCreationModel";  

export class CountryCreationModelMapper {
    public static Map(countryCreationRequest: CountryCreationRequest, countryCreationModel: CountryCreationModel) {
        if(!countryCreationModel)
            return;  

        countryCreationRequest.setName(countryCreationModel.name);
        countryCreationRequest.setDescription(countryCreationModel.description);
        countryCreationRequest.setAnthem(countryCreationModel.anthem);
        countryCreationRequest.setCapitalcity(countryCreationModel.capitalCity);
        countryCreationRequest.setFlaguri(countryCreationModel.flagUri);
        countryCreationRequest.setLanguagesList(countryCreationModel.languages);
    }  

    public static Maps(countriesCreationRequest: CountriesCreationRequest, countriesCreationModel: CountryCreationModel[]) {
        if(!countriesCreationModel)
            return;  

        countriesCreationModel.map(x => {
            let countryCreationRequest = new CountryCreationRequest();
            CountryCreationModelMapper.Map(countryCreationRequest, x);
            countriesCreationRequest.addCountries(countryCreationRequest);
        });
    }
}
```
```ts
import { CountryReply } from "../generated/country.shared_pb";
import { CountryModel } from "../models/countryModel";  

export class CountryReplyMapper {
    public static Map(country: CountryModel, countryReply: CountryReply.AsObject) {
        if(country == null || countryReply == null)
            return;  

        country.id = countryReply.id;
        country.name = countryReply.name;
        country.description = countryReply.description;
        country.capitalCity = countryReply.capitalcity;
        country.flagUri = countryReply.flaguri;
        country.anthem = countryReply.anthem;
        country.languages = countryReply.languagesList;
    }
}
```

[^1]:В этой главе свойство `transport` использоваться не будет, поэтому интересующихся отсылаем к [документации](https://test-focus8.kontur.ru/route/master/cmp-1316-empty-guids)