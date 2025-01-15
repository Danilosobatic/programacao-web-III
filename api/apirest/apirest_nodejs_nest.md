# CRUD com API RESTful usando Node.js e NestJS

Este é um exemplo de como criar uma API RESTful com operações CRUD (Create, Read, Update, Delete) usando **Node.js** e o framework **NestJS**. Vamos criar um servidor básico e implementar as rotas para gerenciar recursos de usuários.

### Estrutura do Projeto

/crud-api-nestjs ├── /src │ ├── /users │ │ ├── user.controller.ts │ │ ├── user.service.ts │ │ └── user.module.ts ├── /node_modules ├── main.ts ├── package.json └── tsconfig.json

 


## Passos para criar o projeto:

### 1. Instalar o NestJS CLI

Primeiro, instale o NestJS CLI globalmente (se ainda não tiver feito isso):

```bash
npm i -g @nestjs/cli
```
###  2. Criar o projeto

Crie um novo projeto NestJS:

```bash
nest new crud-api-nestjs

```

Escolha "npm" como gerenciador de pacotes quando solicitado.


###  3. Instalar dependências necessárias
Dentro do diretório do projeto, instale as dependências:

```bash
 
cd crud-api-nestjs
npm install

```


## Código de Exemplo:
### 1. main.ts - Arquivo principal
Este é o arquivo principal que inicializa a aplicação NestJS.

```typescript
 
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000); // A aplicação vai rodar na porta 3000
}
bootstrap();

```
### 2. user.controller.ts - Controlador
Este arquivo define as rotas e os métodos que serão responsáveis pelas operações CRUD para os usuários.

```typescript
 
import { Controller, Get, Post, Param, Body, Put, Delete } from '@nestjs/common';
import { UserService } from './user.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Controller('users') // Rota principal para o controlador de usuários
export class UserController {
  constructor(private readonly userService: UserService) {}

  // Rota para criar um novo usuário (POST)
  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto);
  }

  // Rota para obter todos os usuários (GET)
  @Get()
  findAll() {
    return this.userService.findAll();
  }

  // Rota para obter um usuário específico (GET by ID)
  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.userService.findOne(id);
  }

  // Rota para atualizar um usuário (PUT)
  @Put(':id')
  update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
    return this.userService.update(id, updateUserDto);
  }

  // Rota para excluir um usuário (DELETE)
  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.userService.remove(id);
  }
}

```
### 3. user.service.ts - Serviço
Este arquivo contém a lógica de negócios, onde a manipulação dos dados (operações CRUD) é realizada. Ele simula a interação com um banco de dados.

``` typescript
 
import { Injectable, NotFoundException } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

// Simulando um banco de dados com um array de usuários
let users = [
  { id: '1', name: 'John Doe', email: 'john.doe@example.com' },
  { id: '2', name: 'Jane Smith', email: 'jane.smith@example.com' },
];

@Injectable()
export class UserService {
  // Criar um novo usuário
  create(createUserDto: CreateUserDto) {
    const newUser = { id: (users.length + 1).toString(), ...createUserDto };
    users.push(newUser);
    return newUser;
  }

  // Obter todos os usuários
  findAll() {
    return users;
  }

  // Obter um usuário por ID
  findOne(id: string) {
    const user = users.find(u => u.id === id);
    if (!user) {
      throw new NotFoundException(`User with id ${id} not found`);
    }
    return user;
  }

  // Atualizar um usuário
  update(id: string, updateUserDto: UpdateUserDto) {
    const userIndex = users.findIndex(u => u.id === id);
    if (userIndex === -1) {
      throw new NotFoundException(`User with id ${id} not found`);
    }
    users[userIndex] = { ...users[userIndex], ...updateUserDto };
    return users[userIndex];
  }

  // Excluir um usuário
  remove(id: string) {
    const userIndex = users.findIndex(u => u.id === id);
    if (userIndex === -1) {
      throw new NotFoundException(`User with id ${id} not found`);
    }
    users.splice(userIndex, 1);
    return { message: `User with id ${id} deleted successfully` };
  }
}
```
### 4. user.module.ts - Módulo
O módulo é responsável por agrupar o controlador e o serviço, além de exportá-los para serem usados na aplicação principal.

``` typescript
 
import { Module } from '@nestjs/common';
import { UserController } from './user.controller';
import { UserService } from './user.service';

@Module({
  controllers: [UserController],
  providers: [UserService],
})
export class UserModule {}

```
### 5. dto/create-user.dto.ts - DTO para criação
Este arquivo define o Data Transfer Object (DTO) usado para a criação de usuários.

``` typescript
 
export class CreateUserDto {
  name: string;
  email: string;
}
```
### 6. dto/update-user.dto.ts - DTO para atualização
Este arquivo define o DTO usado para a atualização de usuários.

``` typescript
 
export class UpdateUserDto {
  name?: string;
  email?: string;
}
```
##Como Executar:
*1. Criar e rodar o servidor:
No terminal, execute o seguinte comando para rodar a aplicação:

``` bash
 
npm run start
``` 
*2.Testar a API:
A API estará disponível em http://localhost:3000/users.

Você pode testar as operações CRUD utilizando ferramentas como Postman, Insomnia, ou curl.

**POST /users - Criar um novo usuário

Exemplo de corpo da requisição:
``` json
 
{
  "name": "Alice Brown",
  "email": "alice.brown@example.com"
}
``` 
**GET /users - Obter todos os usuários
 
**GET /users/:id - Obter um usuário específico

Exemplo de URL: http://localhost:3000/users/1
**PUT /users/:id - Atualizar um usuário

Exemplo de corpo da requisição:
```json
 ` 
{
  "name": "Alice B.",
  "email": "alice.b@example.com"
}
``` 
**DELETE /users/:id - Deletar um usuário

## Conclusão
Este é um exemplo básico de como criar uma API RESTful usando Node.js e NestJS com operações CRUD. O código segue a estrutura recomendada do NestJS, onde o controlador, serviço e módulos são separados, promovendo a organização e escalabilidade do código.

Com o NestJS, você pode facilmente expandir a aplicação para incluir autenticação, validação, persistência de dados em banco de dados real (como MongoDB ou PostgreSQL), e muitos outros recursos.
