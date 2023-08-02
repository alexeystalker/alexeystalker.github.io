---
share: true
tags:
 - gRPC/gRPC-web
---
# Загружаем файл с данными при помощи TypeScript, Web Worker и gRPC-web
Начнем с написания формы и таблицы для отображения стран нашей вики. Редактировать будем компонент *app* (`app.component.html` и `app.component.ts`), так как это точка входа нашего приложения. Учтите, что наша книжка не про Angular, так что код будет простым, чтобы его могли понять даже те, кто не пользуется Angular.
Первой частью нашего компонента будет простая форма для загрузки файла JSON, валидировать который будет код на TypeScript. Вторая часть будет отображать список стран в виде таблицы со всеми их свойствами и двумя кнопками — для изменения и для удаления.
Вот содержимое файла `app.component.html`.
```html
<div class="text-center">
  <h1>CountryWiki.Angular</h1>
</div>

<div class="container mb-5 mt-5 text-center w-25">
  <h2>Upload countries (JSON only): </h2>
  <div class="mb-1 mt-1">
    <input class="form-control" type="file" (change)="onChange($event)" />
  </div>
  <div class="mb-1 mt-1">
    <button (click)="onUpload()" class="btn btn-success">Upload</button>
  </div>
  <div *ngIf="uploadResult !== null && !uploadResult.success" class="mb-2 mt-2 text-danger">
    {{uploadResult.errorMessage}}
  </div>
  <div *ngIf="(uploadResult !== null && uploadResult.isProcessing) || isUploading" class="mb-2 mt-2 text-center text-danger">
    <h2>A file upload is in progress...</h2>
  </div>
</div>
<div class="container text-center">
  <table class="table">
    <thead>
      <tr>
        <th>ID</th>
        <th>Name</th>
        <th>Description</th>
        <th>Capital City</th>
        <th>Anthem</th>
        <th>Spoken languages</th>
        <th>Flag</th>
        <th>Edit</th>
        <th>Delete</th>
      </tr>
    </thead>
    <tbody>
      <tr *ngFor="let country of countries">
        <td>{{country.id}}</td>
        <td>{{country.name}}</td>
        <td>{{country.description}}</td>
        <td>{{country.capitalCity}}</td>
        <td>{{country.anthem}}</td>
        <td>{{country.languages.join(', ')}}</td>
        <td><img src="{{country.flagUri}}" alt="{{country.name}}" height="25" width="25" /></td>
        <td><button (click)="onUpdate(country.id)" class="btn btn-secondary">Update</button></td>
        <td><button (click)="onDelete(country)" class="btn btn-danger">Delete</button></td>
      </tr>
    </tbody>
  </table>
</div>

<router-outlet></router-outlet>
```
Код компонента тоже довольно прост.
```ts
import { Component, OnChanges, OnInit, SimpleChanges } from '@angular/core';
import { UploadResultModel } from './models/uploadResultModel';
import { CountryModel } from './models/countryModel';
import { CountryService } from './services/countryService';
import { Router } from '@angular/router';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  public title = 'CountryWiki.Angular';
  public countries: CountryModel[] = [];
  public errorMessage!: string;
  public uploadResult!: UploadResultModel;
  public isUploading: boolean = false;
  private _file!: File;

  constructor(private _countryService: CountryService, private _router: Router) {

  }

  public ngOnInit() {
    this._countryService.GetAll(this.countries);
  }

  public onChange(event: Event): void {
    const target = event.target as HTMLInputElement;
    this._file = (target.files as FileList)[0];
  }

  public onUpload(): void {
    this.isUploading = this._file != null;
    const worker = new Worker(new URL('./workers/upload-file.worker', import.meta.url));
    worker.onmessage = ({ data }) => {
      this.uploadResult = data;
      if (this.uploadResult.success) {
        this._countryService.Create(
          this.uploadResult.payload,
          this.uploadResult,
          () => {
            this.countries = [];
            this._countryService.GetAll(this.countries);
          });
      } else {
        this.uploadResult.isProcessing = false;
      }
      this.isUploading = false;
      this._router.navigate(['']);
    };
    worker.postMessage(this._file);
  }

  public onUpdate(id: number): void {
    this._router.navigate(['edit', id]);
  }

  public onDelete(country: CountryModel): void {
    this._countryService.Delete(country.id);
    this.countries = this.countries.filter(c => c !== country);
  }
}
```
Метод `ngOnInit()`  заполняет список стран, а метод `onChange()` получает загруженный файл. Метод `onUpdate()` перенаправляет на страницу редактирования, а метод `onDelete()` удаляет страну из списка стран, не вызывая при этом `GetAll()`. Метод `onUpload()` получает файл и отправляет веб-воркеру через `postMessage`. Веб-воркер провалидирует и распарсит файл (мы вернемся к этому чуть позже), после чего вернет результат валидации в коллбэке `onmessage` вместе со списком стран из файла, готовым отправиться на сервер. Если результатом будет успех (`uploadResult.success`), мы вызовем метод `CountryService.Create()` и обновим список стран для таблице в коллбэке этого метода.

Теперь реализуем веб-воркер (Web Worker). Разместим реализацию в файле **src/app/workers/upload-file.worker.ts**.
```ts
/// <reference lib="webworker" />

import { CountryCreationModel } from "../models/countryCreationModel";
import { UploadResultModel } from "../models/uploadResultModel";

addEventListener('message', ({ data }) => {
    const file = data as File;
    const reader = new FileReader();
    let result: UploadResultModel = {
        success: false,
        errorMessage: "",
        payload: [],
        isProcessing: true
    };

    let ext = file.name.substr(file.name.lastIndexOf('.') + 1);
    if (ext === "json" && file.type === "application/json") {
        // Читаем файл
        reader.onloadend = e => {
            try {
                let content = JSON.parse(reader.result!.toString());
                result.success = true;
                result.payload = (content as any[]).map(x => new CountryCreationModel(x));
            }
            catch {
                result.errorMessage = "Cannot parse the file or the file is empty";
            }
            finally {
                postMessage(result);
            }
        };
        reader.readAsText(file);
    }
    else {
        result.errorMessage = "Only JSON files are allowed";
        postMessage(result);
    }
});
```
Воркер асинхронно читает загруженный файл (событие `onloadend` срабатывает единожды после прочтения всего файла), валидирует его формат и парсит содержимое, возвращая в случае успеха массив объектов `CountryCreationModel`. Как уже [[ch-12-write-data-access-with-improbables-grpc-web-client|упоминалось здесь]], при использовании `ts-json-object` требуется обработка ошибок, так что мы обернули создание объектов в `try/catch`. Вы спросите, почему не использовать клиент gRPC-web в веб-воркере? К сожалению, клиент gRPC-web использует для своей работы объект `Windows`, а этот объект недоступен в веб-воркере.