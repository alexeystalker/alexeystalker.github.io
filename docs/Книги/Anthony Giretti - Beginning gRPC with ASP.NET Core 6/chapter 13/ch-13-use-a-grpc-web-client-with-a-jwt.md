---
share: true
tags:
 - security/OIDC
 - gRPC/gRPC-web
---
# Используем клиент gRPC-web с JWT
Как добавить использование токена в клиент gRPC-web на TypeScript? Improbable предоставляет класс `Metadata`, в объект которого мы добавим тот же заголовок `authorization` и значение `bearer {token}`, после чего передадим в запрос:
```ts
public GetAll(countries: CountryModel[], token: String): void {
const metadata = new grpc.Metadata();
metadata.set("authorization", 'bearer ${token}');
grpc.invoke(CountryServiceBrowser.GetAll,
  {
	request: new Empty(),
	host: environment.host,
	metadata: metadata,
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
```