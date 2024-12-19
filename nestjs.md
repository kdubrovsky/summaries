# Nest JS
[Документация](https://docs.nestjs.com)

[Использование CLI](https://docs.nestjs.com/cli/usages)

---

### Глобальная установка
```bash
npm install -g @nestjs/cli
```

### Создание проекта
```bash
npm new <project>
```

### Создание файлов сущностей
```bash
nest g <schematic> <name> <options>

# Schematics:
# class, controller, decorator, filter, gateway, guard, interface, interceptor, middleware, module, pipe, provider, resolver, resource, service
```

## Контроллеры
Суть — получить запрос и отдать ответ. Контроллер не занят бизнес-логикой. Он анализирует запрос и отдает его на обработку нужному сервису. Сервис сформирует ответ и контроллер его вернет клиенту.

### Файл
```
name.contoller.ts
```

### Создание
```bash
nest g controller [name]
```


### Структура
```ts
@Controller('name')

export class CatsController {
	@Get()
	method1(): ResponseType {
		...
		return response;
  }

 	@Post()
	method2(): ResponseType {
		...
		return response;
   }

   ...
}
```

### Формирование адреса эндпоинтов

В `@Controller('segment')` передается префикс-сегмент URL для API.

В декораторы методов контроллера `@Get('path')`, `@Post('path')` передаются пути _после_ этого сегмента

В итоге каждый метод доступен по адресу `[globalapiurl]/segment/path`

Пути в декораторах поддерживают паттерны.

### Формирование ответа API

Код ответа по умолачнию `200`, для `POST` — `201`. Код ответа меняется с помощью декоратора: `@HttpCode(204)`

Простые типы возвращаются как есть, а объекты как JSON.

### Декораторы методов

`@Get()`, `@Post()`, `@Put()`, `@Delete()`, `@Patch()`, `@Options()`, `@Head()`, `@All()`

### Доступ к объекту запроса

Можно получить весь объект запроса обернув в декоратор аргумент метода-обработчика запроса:
```ts
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express'; // ! Этот тип из Express !

@Controller('users')
export class UsersController {

@Get()
  findAll(@Req() request: Request): string {
  ...
  }
```
