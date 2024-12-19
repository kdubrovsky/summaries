# Контроллеры Nest JS
Суть — получить запрос и отдать ответ. Контроллер не занят бизнес-логикой. Он анализирует запрос и отдает его на обработку нужному сервису. Сервис сформирует ответ и контроллер его вернет клиенту.

Контроллер — это класс с методами с точки зрения TS. Методы декорирутся для того, чтобы определить их назначение и ряд свойств. Методы могут быть асинхронными. Они возвращают промисы и это окей для Nest. Можно еще [потоки RxJS](https://rxjs-dev.firebaseapp.com/guide/observable) возвращать.

## Файл
```
name.contoller.ts
```

## Создание
```bash
nest g controller [name]
```

## Интеграция в модуль
Контроллер встраивается в модуль. Он туда импортируется и добавляется в список контроллеров модуля:

```ts
@Module({
  controllers: [UsersController],
})
export class AppModule {}
```

## Структура
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

## Настройка контроллера

В `@Controller('segment')` передается префикс-сегмент URL для API.

Туда можно передать домен целиком чтобы сделать контроллер специфичным для хоста. Например так — `@Controller({ host: 'admin.example.com' })`. В домене даже могут быть динамические значения:

```ts
@Controller({ host: ':account.example.com' })
export class AccountController {
  @Get()
  getInfo(@HostParam('account') account: string) {
    return account;
  }
}
```

## Формирование адреса эндпоинтов

В декораторы методов контроллера `@Get('path')`, `@Post('path')` передаются пути _после_ префикса уровня контроллера.

В итоге каждый метод доступен по адресу `[globalapiurl]/segment/path`

Пути в декораторах поддерживают паттерны.

Если мы хотим получить параметр как часть пути, то эта динамическая часть указывается с помощью двоеточия:

```ts
@Get(':id')
findOne(@Param() params: any): string {
  console.log(params.id);
  ...
}

// или так

@Get(':id')
findOne(@Param('id') id: string): string {
	console.log(id);
	...
}

// так мы будет получать ответ по адресам вида /api/product/123
// получая 123 как id
```

## Формирование ответа API

Код ответа по умолачнию `200`, для `POST` — `201`. Код ответа меняется с помощью декоратора: `@HttpCode(204)`.

Можно поменять заголовки ответа с помощью декоратора для метода `@Header(<key>, <value>)`

Простые типы возвращаются как есть, а объекты как JSON.

Можно сделать редирект с помощью декоратора для метода `@Redirect(<url>, <statucCode>)` или сделать это динамически вернув соответствующий объект типа `HttpRedirectResponse`.

## Декораторы для методов

`@Get()`, `@Post()`, `@Put()`, `@Delete()`, `@Patch()`, `@Options()`, `@Head()`, `@All()`

## Доступ к объекту запроса

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

[Описание свойств Request](https://expressjs.com/en/api.html#req)

Помимо этого можно частично вытащить то, что нужно из запроса:

| **Декоратор**              | **Что отдает**                           |
|----------------------------|------------------------------------------|
| `@Response()`, `@Res()`    | `res` (Library-specific mode)            |
| `@Next()`                  | `next`                                   |
| `@Session()`               | `req.session`                            |
| `@Param(key?: string)`     | `req.params` / `req.params[key]`         |
| `@Body(key?: string)`      | `req.body` / `req.body[key]`             |
| `@Query(key?: string)`     | `req.query` / `req.query[key]`           |
| `@Headers(name?: string)`  | `req.headers` / `req.headers[name]`      |
| `@Ip()`                    | `req.ip`                                 |
| `@HostParam()`             | `req.hosts`                              |

## Доступ к телу запроса

Мы можем не просто частично получить объект запроса с помощью декоратора `@Body()`, но и указать DTO-класс в качестве типа для содержимого `body`.

Для этого мы создаем в отдельном файле DTO-класс:

```ts
export class CreateUserDto {
  name: string;
  age: number;
  job: string;
}
```

И указываем его как тип для параметра метода который оперирует телом запроса:

```ts
@Post()
async create(@Body() createCatDto: CreateUserDto) {
	...
}
```

## Доступ к объекту ответа

Nest позволяет методам контроллера получить доступ к объекту ответа и в ручном режиме на уровне Express управлять ответом. Если вы использовали @Res() в атрибутах метода, то вы переши в так называемый Library-specific mode (чаще всего это Express). Это опасное поведение так как прекращается функционирование дефолтных вещей предоставляемых Nest.

```ts
import { Controller, Get, Post, Res, HttpStatus } from '@nestjs/common';
import { Response } from 'express';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Res() res: Response) {
    res.status(HttpStatus.CREATED).send();
  }

  @Get()
  findAll(@Res() res: Response) {
     res.status(HttpStatus.OK).json([]);
  }
}
```
