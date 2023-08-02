---
share: true
tags:
 - gRPC/gRPC-web
---
# Управляем данными при помощи TypeScript и gRPC-web
Наш компонент для редактирования будет позволять изменять описание страны. Как только мы распознаём измененное описание, оно валидируется и отправляется на сервер в случае успеха.
Чтобы добавить компонент, воспользуемся командой Angular CLI из папки **src/app**:
```bash
ng generate c edit
```
Команда создаст подпапку **edit** и нужные файлы в ней, нужно только заполнить их своей реализацией.
Вот реализация `edit.component.html`:
```html
<div class="container mb-5 mt-5 text-center">
    <h2>Edit Country - {{country.name}}</h2>
    <div *ngIf="country">
        <div>
            <textarea rows="4" cols="50" name="description" [ngModel]="country.description" (ngModelChange)="onChange($event)">
                {{country.description}}
            </textarea>
        </div>
        <div *ngIf="actionResult !== null && !actionResult.success" class="text-danger">
            {{actionResult.errorMessage}}
        </div>
        <div>
            <button (click)="onUpdate()">Update</button>
        </div>
    </div>
</div>
```
А вот реализация `edit.component.ts`:
```ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { ActionResultModel } from '../models/actionResultModel';
import { CountryModel } from '../models/countryModel';
import { CountryUpdateModel } from '../models/countryUpdateModel';
import { CountryService } from '../services/countryService';

@Component({
  selector: 'app-edit',
  templateUrl: './edit.component.html',
  styleUrls: ['./edit.component.css']
})
export class EditComponent implements OnInit {
  public country: CountryModel = new CountryModel();
  public actionResult: ActionResultModel = new ActionResultModel();
  private static readonly _errorValidationMessage = "Description must be between 20 and 200 characters";

  constructor(private _countryService: CountryService,
    private _route: ActivatedRoute,
    private _router: Router) {
    }

  ngOnInit(): void {
    this._route.params.subscribe(p => {
      let id = p["id"] as number;
      this._countryService.Get(id, this.country);
    });
  }

  public onChange(event: Event): void {
    let description = event as unknown as string;
    this.country.description = description;
  }

  public onUpdate(): void {
    if (this.country.description.length > 200 || this.country.description.length < 20) {
      this.actionResult = <ActionResultModel>({
        success: false,
        errorMessage: EditComponent._errorValidationMessage
      });
    }
    else {
      this.actionResult = <ActionResultModel>({
        success: true,
        errorMessage: ""
      });
      this._countryService.Update(<CountryUpdateModel>({
        id: this.country.id,
        description: this.country.description
      }));
      this._router.navigate(['']).then(() => {
        window.location.reload();
      })
    }
  }
}
```
Здесь в методе `ngOnInit` мы перехватываем параметр `id` маршрута, используем его, чтобы получить объект страны при помощи `_countryService.Get()`. Метод `onChange()` используется для распознавания изменений в поле `description`. Изменения сохраняются в методе `onUpdate()`, который срабатывает по нажатию на кнопку “Update”.