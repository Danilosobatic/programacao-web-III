# CRUD Node.js, Angular e SQLite

Este projeto implementa um CRUD utilizando **Node.js** no back-end, **Angular** no front-end, e **SQLite** como banco de dados.

## Estrutura do Projeto

crud-node-angular-sqlite/ │ ├── backend/ # Diretório para o back-end (Node.js) │ ├── node_modules/ # Dependências do back-end │ ├── server.js # Arquivo principal do servidor Node.js │ ├── database.js # Arquivo para configurar o banco de dados SQLite │ └── package.json # Dependências do back-end │ └── frontend/ # Diretório para o front-end (Angular) ├── node_modules/ # Dependências do front-end ├── src/ │ ├── app/ │ │ ├── app.component.ts # Componente principal do Angular │ │ ├── app.module.ts # Módulo principal do Angular │ │ └── user.service.ts # Serviço para comunicação com o back-end │ └── index.html # Arquivo HTML principal └── angular.json # Configurações do Angular

 
## Passo 1: Criar o Back-End (Node.js e SQLite)

### 1.1. Inicializar o Projeto Node.js

1. Crie uma pasta para o back-end:

    ```bash
    mkdir backend
    cd backend
    npm init -y
    npm install express sqlite3 body-parser cors
    ```

2. Crie o arquivo `server.js` no diretório `backend` com o seguinte conteúdo:

    ```js
    const express = require('express');
    const sqlite3 = require('sqlite3').verbose();
    const cors = require('cors');
    const bodyParser = require('body-parser');

    const app = express();
    const port = 3000;

    // Configuração do banco de dados SQLite
    const db = new sqlite3.Database('./database.db', (err) => {
        if (err) {
            console.error('Erro ao conectar ao banco de dados SQLite:', err);
        } else {
            console.log('Conectado ao banco de dados SQLite');
        }
    });

    // Criar a tabela se não existir
    db.run(`
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            email TEXT NOT NULL
        );
    `);

    app.use(cors());
    app.use(bodyParser.json());

    // Rota para obter todos os usuários
    app.get('/users', (req, res) => {
        db.all('SELECT * FROM users', [], (err, rows) => {
            if (err) {
                return res.status(500).send(err);
            }
            res.json(rows);
        });
    });

    // Rota para obter um usuário pelo ID
    app.get('/users/:id', (req, res) => {
        const { id } = req.params;
        db.get('SELECT * FROM users WHERE id = ?', [id], (err, row) => {
            if (err) {
                return res.status(500).send(err);
            }
            if (!row) {
                return res.status(404).send('Usuário não encontrado');
            }
            res.json(row);
        });
    });

    // Rota para criar um novo usuário
    app.post('/users', (req, res) => {
        const { name, email } = req.body;
        db.run('INSERT INTO users (name, email) VALUES (?, ?)', [name, email], function(err) {
            if (err) {
                return res.status(500).send(err);
            }
            res.status(201).json({ id: this.lastID, name, email });
        });
    });

    // Rota para atualizar um usuário
    app.put('/users/:id', (req, res) => {
        const { id } = req.params;
        const { name, email } = req.body;
        db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id], function(err) {
            if (err) {
                return res.status(500).send(err);
            }
            if (this.changes === 0) {
                return res.status(404).send('Usuário não encontrado');
            }
            res.json({ id, name, email });
        });
    });

    // Rota para excluir um usuário
    app.delete('/users/:id', (req, res) => {
        const { id } = req.params;
        db.run('DELETE FROM users WHERE id = ?', [id], function(err) {
            if (err) {
                return res.status(500).send(err);
            }
            if (this.changes === 0) {
                return res.status(404).send('Usuário não encontrado');
            }
            res.status(204).send();
        });
    });

    app.listen(port, () => {
        console.log(`Servidor rodando na porta ${port}`);
    });
    ```

### 1.2. Criar o Banco de Dados SQLite

Não é necessário criar explicitamente o banco de dados. O arquivo será criado automaticamente quando o servidor for executado pela primeira vez. O banco de dados será armazenado no arquivo `database.db`.

Agora, execute o servidor:

```bash
node server.js


```

## Passo 2: Criar o Front-End (Angular)

### 2.1. Criar o Projeto Angular

1. Crie um novo projeto Angular:

    ```bash
    ng new frontend
    cd frontend
    ```

2. Instale o `HttpClient` para realizar as requisições HTTP:

    ```bash
    npm install @angular/common/http
    ```

### 2.2. Criar o Serviço `user.service.ts`

Crie o serviço `user.service.ts` dentro do diretório `src/app` para fazer a comunicação com o back-end. Esse serviço usará o `HttpClient` para consumir a API do Node.js.

Crie o arquivo `user.service.ts` com o seguinte conteúdo:

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

  // Obter todos os usuários
  getUsers(): Observable<any> {
    return this.http.get(this.apiUrl);
  }

  // Obter um usuário pelo ID
  getUser(id: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/${id}`);
  }

  // Adicionar um novo usuário
  addUser(user: any): Observable<any> {
    return this.http.post(this.apiUrl, user);
  }

  // Atualizar um usuário
  updateUser(id: number, user: any): Observable<any> {
    return this.http.put(`${this.apiUrl}/${id}`, user);
  }

  // Excluir um usuário
  deleteUser(id: number): Observable<any> {
    return this.http.delete(`${this.apiUrl}/${id}`);
  }
}
```


###  2.3. Atualizar o Componente Principal app.component.ts
O próximo passo é atualizar o componente principal (app.component.ts) para lidar com as operações de CRUD no front-end. Esse componente usará o UserService para buscar, adicionar, editar e excluir usuários.

Crie o arquivo app.component.ts com o seguinte conteúdo:

```ts
Copiar código
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

  // Carregar todos os usuários
  loadUsers() {
    this.userService.getUsers().subscribe(data => {
      this.users = data;
    });
  }

  // Adicionar um novo usuário
  addUser() {
    if (this.newUser.name && this.newUser.email) {
      this.userService.addUser(this.newUser).subscribe(() => {
        this.loadUsers();
        this.newUser = { name: '', email: '' }; // Limpar os campos
      });
    }
  }

  // Preparar a edição de um usuário
  edit(id: number) {
    this.userService.getUser(id).subscribe(user => {
      this.editUser = { ...user };
    });
  }

  // Atualizar um usuário
  updateUser() {
    if (this.editUser.name && this.editUser.email) {
      this.userService.updateUser(this.editUser.id, this.editUser).subscribe(() => {
        this.loadUsers();
        this.editUser = { id: 0, name: '', email: '' }; // Limpar os campos
      });
    }
  }

  // Excluir um usuário
  deleteUser(id: number) {
    this.userService.deleteUser(id).subscribe(() => {
      this.loadUsers();
    });
  }
}
```
### 2.4. Atualizar o HTML app.component.html
Agora, crie o arquivo app.component.html para criar a interface de usuário onde será possível listar, adicionar, editar e excluir usuários.

Crie o arquivo app.component.html com o seguinte conteúdo:

```html
Copiar código
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
### 2.5. Configurar o Módulo app.module.ts
Agora, configure o módulo app.module.ts para incluir o HttpClientModule e o FormsModule, necessários para fazer as requisições HTTP e habilitar o two-way data binding.

Crie o arquivo app.module.ts com o seguinte conteúdo:

```ts
Copiar código
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
###  2.6. Rodando o Front-End
Execute o servidor Angular:

```bash
Copiar código
ng serve
```
Acesse a aplicação em seu navegador:

Navegue até http://localhost:4200 para interagir com o front-end da aplicação. Você poderá adicionar, listar, editar e excluir usuários.