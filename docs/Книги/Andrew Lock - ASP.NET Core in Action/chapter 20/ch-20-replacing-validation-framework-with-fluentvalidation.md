---
share: true
tags:
 - NET/ASPNETCore/validation
---
# Замена фреймворка валидации на FluentValidation
[[library-fluentvalidation|FluentValidation]] — альтернативный фреймворк для валидации. С ним код валидации пишется отдельно от кода [[binding-model|модели привязки]]. Преимущества:
- нет ограничений атрибутов, таких как проблема [[ch-20-building-custom-validation-attribute|внедрения зависимостей]];
- намного проще создавать правила, применяющиеся к нескольким свойствам (например, чтобы значение `EndDate`  было позже, чем значение `StartDate`);
- как правило, тестировать валидаторы FluentValidation проще;
- валидация строго типизирована;
- отделение логики валидации от модели, возможно, лучше соответствует [[single-responsibility-principle|принципу единой ответственности]].

Прежде, чем добавить FluentValidation в приложение, рассмотрим, как выглядят валидаторы FluentValidation.
#### [[ch-20-comparing-fluentvalidation-to-dataannotations-attributes|Сравнение FluentValidation и атрибутов DataAnnotations]]
#### [[ch-20-adding-fluentvalidation-to-your-application|Добавляем FluentValidation в приложение]]