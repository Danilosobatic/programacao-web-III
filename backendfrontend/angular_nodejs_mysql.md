# CRUD Node.js, Angular e MySQL

Este projeto cria um CRUD simples utilizando **Node.js** no back-end, **Angular** no front-end, e **MySQL** como banco de dados.

## Estrutura do Projeto

```
crud-node-angular-mysql/
│
├── backend/                # Diretório para o back-end (Node.js)
│   ├── node_modules/    # Dependências do back-end
│   ├── server.js        # Arquivo principal do servidor Node.js
│   └── package.json     # Dependências do back-end
│
└── frontend/               # Diretório para o front-end (Angular)
    ├── node_modules/       # Dependências do front-end
    ├── src/                # Código-fonte do Angular
    │   ├── app/           
    │   │   ├── app.component.ts  # Componente principal do Angular
    │   │   ├── app.module.ts     # Módulo principal do Angular
    │   │   └── user.service.ts   # Serviço para comunicação com o back-end
    │   
    │   └── index.html             # Arquivo HTML principal
    
    └── angular.json          # Configurações do Angular
```



## Passo 1: Criar o Back-End (Node.js e MySQL)

### 1.1. Inicializar o Projeto Node.js

1. Crie uma pasta para o back-end:

    ```bash
    mkdir backend
    cd backend
    npm init -y
    npm install express mysql2 body-parser cors
    ```

2. Crie o arquivo `server.js` no diretório `backend` com o seguinte conteúdo:

    ```js
    const express = require('express');
    const mysql = require('mysql2');
    const cors = require('cors');
    const bodyParser = require('body-parser');

    const app = express();
    const port = 3000;

    // Configuração do MySQL
    const db = mysql.createConnection({
        host: 'localhost',
        user: 'root',
        password: '', // Insira a senha do seu banco de dados MySQL
        database: 'crud_db'
    });

    db.connect(err => {
        if (err) {
            console.error('Erro ao conectar ao banco de dados:', err);
        } else {
            console.log('Conectado ao banco de dados MySQL');
        }
    });

    app.use(cors());
    app.use(bodyParser.json());

    // Rota para obter todos os usuários
    app.get('/users', (req, res) => {
        db.query('SELECT * FROM users', (err, results) => {
            if (err) {
                return res.status(500).send(err);
            }
            res.json(results);
        });
    });

    // Rota para obter um usuário pelo ID
    app.get('/users/:id', (req, res) => {
        const { id } = req.params;
        db.query('SELECT * FROM users WHERE id = ?', [id], (err, results) => {
            if (err) {
                return res.status(500).send(err);
            }
            if (results.length === 0) {
                return res.status(404).send('Usuário não encontrado');
            }
            res.json(results[0]);
        });
    });

    // Rota para criar um novo usuário
    app.post('/users', (req, res) => {
        const { name, email } = req.body;
        db.query('INSERT INTO users (name, email) VALUES (?, ?)', [name, email], (err, results) => {
            if (err) {
                return res.status(500).send(err);
            }
            res.status(201).json({ id: results.insertId, name, email });
        });
    });

    // Rota para atualizar um usuário
    app.put('/users/:id', (req, res) => {
        const { id } = req.params;
        const { name, email } = req.body;
        db.query('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id], (err, results) => {
            if (err) {
                return res.status(500).send(err);
            }
            if (results.affectedRows === 0) {
                return res.status(404).send('Usuário não encontrado');
            }
            res.json({ id, name, email });
        });
    });

    // Rota para excluir um usuário
    app.delete('/users/:id', (req, res) => {
        const { id } = req.params;
        db.query('DELETE FROM users WHERE id = ?', [id], (err, results) => {
            if (err) {
                return res.status(500).send(err);
            }
            if (results.affectedRows === 0) {
                return res.status(404).send('Usuário não encontrado');
            }
            res.status(204).send();
        });
    });

    app.listen(port, () => {
        console.log(`Servidor rodando na porta ${port}`);
    });
    ```

### 1.2. Criar o Banco de Dados MySQL

1. No MySQL, crie o banco de dados e a tabela para armazenar os usuários. Use o seguinte comando:

    ```sql
    CREATE DATABASE crud_db;

    USE crud_db;

    CREATE TABLE users (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(100),
        email VARCHAR(100)
    );
    ```

2. Agora, execute o servidor com:

    ```bash
    node server.js
    ```

---

## Passo 2: Criar o Front-End (Angular)

### 2.1. Criar o Projeto Angular

1. Crie um novo projeto Angular:

    ```bash
    ng new frontend
    cd frontend
    ```

2. Instale o `HttpClient` para fazer as requisições HTTP:

    ```bash
    npm install @angular/common/http
    ```

3. Crie o serviço `user.service.ts` dentro do diretório `src/app`:

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

4. Atualize o componente principal `app.component.ts`:

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

5. No arquivo `app.component.html`, crie a interface para listar, adicionar, editar e excluir usuários:

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

2. Acesse `http://localhost:4200` no navegador para interagir com a aplicação.

---

## Conclusão

Você agora tem um CRUD completo com **Node.js** no back-end, **Angular** no front-end, e **MySQL** como banco de dados. O Node.js lida com as requisições RESTful, enquanto o Angular comunica-se com o back-end usando **HTTP** para operações CRUD. O banco de dados MySQL armazena os dados de forma persistente.

## Referências

- [Node.js](https://nodejs.org/)
- [Express](https://expressjs.com/)
- [Angular](https://angular.io/)
- [MySQL](https://www.mysql.com/)
