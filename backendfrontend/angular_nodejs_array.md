# CRUD Node.js e Angular com Array

Este projeto implementa um CRUD simples utilizando **Node.js** para o back-end, **Angular** para o front-end e um **array** para armazenar os dados no back-end.


** Back-end (Node.js)**  – Será responsável por manipular os dados em memória (usando um array) e fornecer as rotas para criar, ler, atualizar e excluir registros.
** Front-end (Angular)**  – Será responsável pela interface do usuário e pela comunicação com o back-end para realizar as operações CRUD.

## Estrutura do Projeto

crud-node-angular/ │ ├── backend/ │ └── server.js │ └── frontend/ ├── src/ │ ├── app/ │ │ ├── app.component.ts │ │ ├── app.module.ts │ │ └── user.service.ts │ └── index.html └── angular.json]



## Passo 1: Criar o Back-End (Node.js)

### 1.1. Inicializando o Projeto Node.js

1. Crie uma pasta para o back-end e inicialize o projeto Node.js:

    ```bash
    mkdir backend
    cd backend
    npm init -y
    npm install express body-parser cors
    ```

2. Crie o arquivo `server.js` com o seguinte conteúdo:

    ```js
    const express = require('express');
    const cors = require('cors');
    const bodyParser = require('body-parser');

    const app = express();
    const port = 3000;

    app.use(cors());
    app.use(bodyParser.json());

    // Array para armazenar os usuários
    let users = [
        { id: 1, name: 'João Silva', email: 'joao@example.com' },
        { id: 2, name: 'Maria Souza', email: 'maria@example.com' }
    ];

    // Rota para obter todos os usuários
    app.get('/users', (req, res) => {
        res.json(users);
    });

    // Rota para obter um usuário pelo ID
    app.get('/users/:id', (req, res) => {
        const user = users.find(u => u.id === parseInt(req.params.id));
        if (user) {
            res.json(user);
        } else {
            res.status(404).send('Usuário não encontrado');
        }
    });

    // Rota para criar um novo usuário
    app.post('/users', (req, res) => {
        const { name, email } = req.body;
        const newUser = {
            id: users.length + 1,
            name,
            email
        };
        users.push(newUser);
        res.status(201).json(newUser);
    });

    // Rota para atualizar um usuário
    app.put('/users/:id', (req, res) => {
        const user = users.find(u => u.id === parseInt(req.params.id));
        if (user) {
            const { name, email } = req.body;
            user.name = name;
            user.email = email;
            res.json(user);
        } else {
            res.status(404).send('Usuário não encontrado');
        }
    });

    // Rota para excluir um usuário
    app.delete('/users/:id', (req, res) => {
        const userIndex = users.findIndex(u => u.id === parseInt(req.params.id));
        if (userIndex !== -1) {
            users.splice(userIndex, 1);
            res.status(204).send();
        } else {
            res.status(404).send('Usuário não encontrado');
        }
    });

    app.listen(port, () => {
        console.log(`Servidor rodando na porta ${port}`);
    });
    ```

3. Execute o servidor:

    ```bash
    node server.js
    ```

Isso inicializa o servidor Node.js na porta `3000`, pronto para aceitar as requisições do front-end.

---

## Passo 2: Criar o Front-End (Angular)

### 2.1. Criando o Projeto Angular

1. Crie uma nova aplicação Angular:

    ```bash
    ng new frontend
    cd frontend
    ```

2. Instale o serviço HTTP para fazer requisições ao back-end:

    ```bash
    npm install @angular/common/http
    ```

3. Crie o serviço para gerenciar as requisições ao back-end. Dentro da pasta `src/app`, crie um arquivo chamado `user.service.ts`:

    ```ts
    import { Injectable } from '@angular/core';
    import { HttpClient } from '@angular/common/http';
    import { Observable } from 'rxjs';

    @Injectable({
      providedIn: 'root'
    })
    export class UserService {
      private apiUrl = 'http://localhost:3000/users';

      constructor(private http: HttpClient) { }

      getUsers(): Observable<any> {
        return this.http.get(this.apiUrl);
      }

      getUser(id: number): Observable<any> {
        return this.http.get(`${this.apiUrl}/${id}`);
      }

      addUser(user: any): Observable<any> {
        return this.http.post(this.apiUrl, user);
      }

      updateUser(id: number, user: any): Observable<any> {
        return this.http.put(`${this.apiUrl}/${id}`, user);
      }

      deleteUser(id: number): Observable<any> {
        return this.http.delete(`${this.apiUrl}/${id}`);
      }
    }
    ```

4. No arquivo `app.component.ts`, utilize o `UserService` para fazer as operações CRUD. O código abaixo vai permitir adicionar, listar, editar e excluir usuários:

    ```ts
    import { Component, OnInit } from '@angular/core';
    import { UserService } from './user.service';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.css']
    })
    export class AppComponent implements OnInit {
      users: any[] = [];
      newUser = { name: '', email: '' };
      editUser = { id: 0, name: '', email: '' };

      constructor(private userService: UserService) {}

      ngOnInit() {
        this.loadUsers();
      }

      loadUsers() {
        this.userService.getUsers().subscribe(data => {
          this.users = data;
        });
      }

      addUser() {
        if (this.newUser.name && this.newUser.email) {
          this.userService.addUser(this.newUser).subscribe(() => {
            this.loadUsers();
            this.newUser = { name: '', email: '' }; // Limpar os campos
          });
        }
      }

      edit(id: number) {
        this.userService.getUser(id).subscribe(user => {
          this.editUser = { ...user };
        });
      }

      updateUser() {
        if (this.editUser.name && this.editUser.email) {
          this.userService.updateUser(this.editUser.id, this.editUser).subscribe(() => {
            this.loadUsers();
            this.editUser = { id: 0, name: '', email: '' }; // Limpar os campos
          });
        }
      }

      deleteUser(id: number) {
        this.userService.deleteUser(id).subscribe(() => {
          this.loadUsers();
        });
      }
    }
    ```

5. No arquivo `app.component.html`, crie a interface para exibir os usuários e permitir a interação CRUD:

    ```html
    <div>
      <h1>CRUD de Usuários</h1>

      <!-- Formulário para adicionar novo usuário -->
      <h3>Adicionar Novo Usuário</h3>
      <input [(ngModel)]="newUser.name" placeholder="Nome" />
      <input [(ngModel)]="newUser.email" placeholder="Email" />
      <button (click)="addUser()">Adicionar</button>

      <!-- Tabela de usuários -->
      <h3>Lista de Usuários</h3>
      <table>
        <tr>
          <th>ID</th>
          <th>Nome</th>
          <th>Email</th>
          <th>Ações</th>
        </tr>
        <tr *ngFor="let user of users">
          <td>{{ user.id }}</td>
          <td>{{ user.name }}</td>
          <td>{{ user.email }}</td>
          <td>
            <button (click)="edit(user.id)">Editar</button>
            <button (click)="deleteUser(user.id)">Excluir</button>
          </td>
        </tr>
      </table>

      <!-- Formulário para editar usuário -->
      <div *ngIf="editUser.id">
        <h3>Editar Usuário</h3>
        <input [(ngModel)]="editUser.name" placeholder="Nome" />
        <input [(ngModel)]="editUser.email" placeholder="Email" />
        <button (click)="updateUser()">Atualizar</button>
      </div>
    </div>
    ```

6. No arquivo `app.module.ts`, importe o `HttpClientModule` e o `FormsModule` para permitir as requisições HTTP e o two-way data binding:

    ```ts
    import { NgModule } from '@angular/core';
    import { BrowserModule } from '@angular/platform-browser';
    import { HttpClientModule } from '@angular/common/http';
    import { FormsModule } from '@angular/forms';
    import { AppComponent } from './app.component';
    import { UserService } from './user.service';

    @NgModule({
      declarations: [
        AppComponent
      ],
      imports: [
        BrowserModule,
        HttpClientModule,
        FormsModule
      ],
      providers: [UserService],
      bootstrap: [AppComponent]
    })
    export class AppModule { }
    ```

### 2.2. Rodando o Front-End

1. Execute o servidor Angular:

    ```bash
    ng serve
    ```

2. Acesse `http://localhost:4200` para interagir com o front-end.

---

## Conclusão

Você agora tem um CRUD completo usando **Node.js** no back-end, **Angular** no front-end, e um **array** simples para armazenar os dados temporariamente no back-end.

A comunicação entre o front-end e o back-end é feita via HTTP, onde o Angular consome as APIs REST fornecidas pelo Node.js. O array no back-end é usado apenas como armazenamento temporário, e em um cenário real, você provavelmente usaria um banco de dados.

## Referências

- [Node.js](https://nodejs.org/)
- [Angular](https://angular.io/)
- [Express](https://expressjs.com/)
- [rxjs](https://rxjs.dev/)
