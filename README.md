# Création d'une app Ionic

On commence sur du blank en spécifiant que l'on soihaite utiliser angular

```bash
ionic start ionic-rest-api-users blank --type=angular
```

Déplacement dans le dossier

```bash
cd ionic-rest-api-users
````

## Génération des pages

```bash
ng generate page pages/create
ng generate page pages/list
ng generate page pages/update
````

On peut aussi utiliser:
```bash
ionic g page pages/create
ionic g page pages/list
ionic g page pages/update
```

## Navigation et routes

Les routes et le code respectif se situe dans **app-routing.ts**:

```js
import { NgModule } from '@angular/core';
import { PreloadAllModules, RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  {
    path: '',
    redirectTo: 'create',
    pathMatch: 'full'
  },
  {
    path: 'create',
    loadChildren: () => import('./pages/create/create.module').then( m => m.CreatePageModule)
  },
  {
    path: 'list',
    loadChildren: () => import('./pages/list/list.module').then( m => m.ListPageModule)
  },
  {
    path: 'update/:id',
    loadChildren: () => import('./pages/update/update.module').then( m => m.UpdatePageModule)
  },
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, { preloadingStrategy: PreloadAllModules })
  ],
  exports: [RouterModule]
})

export class AppRoutingModule { }
```

## Activation de la navigation dans le html avec routerLink

**app.component.html*

```markup
<ion-app>
  <ion-router-outlet></ion-router-outlet>
</ion-app>

<!-- Add navigation -->
<ion-tabs>
  <ion-tab-bar slot="bottom">
    <ion-tab-button routerLinkActive="tab-selected" routerLink="/list" tab="list">
      <ion-icon name="list-outline"></ion-icon>
      <ion-label>Liste des utilisateurs</ion-label>
    </ion-tab-button>

    <ion-tab-button routerLinkActive="tab-selected" routerLink="/create" tab="create">
      <ion-icon name="person-outline"></ion-icon>
      <ion-label>Ajouter un utilisateur</ion-label>
    </ion-tab-button>
  </ion-tab-bar>
</ion-tabs>
```

## Import HttpClientModule dans le module App

Pour utiliser HttpClient afin d'accéder aux méthodes HTTP, il faut importer **HttpClientModule** depuis le paquer **@angular/common/http** mais aussi l'ajouter au tableau des imports dans le fichier **app.module.ts**

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { RouteReuseStrategy } from '@angular/router';

import { IonicModule, IonicRouteStrategy } from '@ionic/angular';

import { AppComponent } from './app.component';
import { AppRoutingModule } from './app-routing.module';

// Import
import { HttpClientModule } from '@angular/common/http';


@NgModule({
  declarations: [AppComponent],
  entryComponents: [],
  imports: [
    BrowserModule, 
    IonicModule.forRoot(), 
    AppRoutingModule,
    HttpClientModule 
  ],
  providers: [{ provide: RouteReuseStrategy, useClass: IonicRouteStrategy }],
  bootstrap: [AppComponent],
})

export class AppModule {}
```

## Create JSON Server

To make HTTP requests, you need a server; consequently, we can take the help of the json-server npm package. Execute command to install the plugin.

```bash
npm i -g json-server
```

Create a folder, name it backend and inside that folder, also create a database.json file. After that, you need to add the test data in the **backend/database.json** file.

```json
{
    "users": [{
        "id": 1,
        "name": "Byron Carlson",
        "email": "byron.carlson@example.com",
        "username": "carlson"
    }, {
        "id": 2,
        "name": "Sebastian Jacobs",
        "email": "sebastian.jacobs@example.com",
        "username": "jacobs"
    }, {
        "id": 3,
        "name": "Cassandra Holland",
        "email": "cassandra.holland@example.com",
        "username": "holland"
    }, {
        "id": 4,
        "name": "Henry Hanson",
        "email": "henry.hanson@example.com",
        "username": "hanson"
    }]
}
```

The users’ data object can now be used for handling CRUD operations through HTTP methods. So, execute the subsequent command to start the dummy server.

```bash
json-server --watch backend/database.json
```

Now, the json server has been started, and you can check the server running on the following urls:

```bash
\{^_^}/ hi!

  Resources
  http://localhost:3000/users

  Home
  http://localhost:3000
```


## Create Ionic Service

Now, we will generate angular service in Ionic, and it will allow you to manage REST API in Ionic with HttpClient API. You can see the example of Ionic HTTP Headers, Ionic Observables, to make Post, Get, Put and Delete requests in Ionic.

Let’s use command on the terminal to create service file:

```bash
ng generate service services/userCrud

ou

ionic g service services/userCrud
```


Import HttpClient, HttpHeaders to send API calls in Ionic via service with Observable’s help, you get a response, and you can handle errors with RxJS catchError and Observable API.

Update **services/userCrud.service.ts** file:

```javascript
import { Injectable } from '@angular/core';

import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Observable, of } from 'rxjs';
import { catchError, tap } from 'rxjs/operators';

export class User {
  _id: number;
  name: string;
  email: string;
  username: string;
}

@Injectable({
  providedIn: 'root'
})

export class UserCrudService {

  endpoint = 'http://localhost:3000/users';

  httpOptions = {
    headers: new HttpHeaders({ 'Content-Type': 'application/json' })
  };

  constructor(private httpClient: HttpClient) { }

  createUser(user: User): Observable<any> {
    return this.httpClient.post<User>(this.endpoint, JSON.stringify(user), this.httpOptions)
      .pipe(
        catchError(this.handleError<User>('Error occured'))
      );
  }

  getUser(id): Observable<User[]> {
    return this.httpClient.get<User[]>(this.endpoint + '/' + id)
      .pipe(
        tap(_ => console.log(`User fetched: ${id}`)),
        catchError(this.handleError<User[]>(`Get user id=${id}`))
      );
  }

  getUsers(): Observable<User[]> {
    return this.httpClient.get<User[]>(this.endpoint)
      .pipe(
        tap(users => console.log('Users retrieved!')),
        catchError(this.handleError<User[]>('Get user', []))
      );
  }

  updateUser(id, user: User): Observable<any> {
    return this.httpClient.put(this.endpoint + '/' + id, JSON.stringify(user), this.httpOptions)
      .pipe(
        tap(_ => console.log(`User updated: ${id}`)),
        catchError(this.handleError<User[]>('Update user'))
      );
  }

  deleteUser(id): Observable<User[]> {
    return this.httpClient.delete<User[]>(this.endpoint + '/' + id, this.httpOptions)
      .pipe(
        tap(_ => console.log(`User deleted: ${id}`)),
        catchError(this.handleError<User[]>('Delete user'))
      );
  }


  private handleError<T>(operation = 'operation', result?: T) {
    return (error: any): Observable<T> => {
      console.error(error);
      console.log(`${operation} failed: ${error.message}`);
      return of(result as T);
    };
  }  
  
}
```

## Ionic Http POST Example

The onSubmit() method makes the API call via the createUser() method, redirect the user to the users’ list page once the request is made.

Open **create.module.ts** file and **import ReactiveFormsModule;** this lets you get along with the angular forms.

```typescript
import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
  imports: [
    ReactiveFormsModule
  ]
})
```

## Add create.page.ts file:

```typescript
import { Component, OnInit, NgZone } from '@angular/core';

import { Router } from '@angular/router';
import { FormGroup, FormBuilder } from "@angular/forms";
import { UserCrudService } from './../../services/user-crud.service';

@Component({
  selector: 'app-create',
  templateUrl: './create.page.html',
  styleUrls: ['./create.page.scss'],
})

export class CreatePage implements OnInit {

  userForm: FormGroup;

  constructor(
    private router: Router,
    public formBuilder: FormBuilder,
    private zone: NgZone,
    private userCrudService: UserCrudService    
  ) {
    this.userForm = this.formBuilder.group({
      name: [''],
      email: [''],
      username: ['']
    })
  }

  ngOnInit() { }

  onSubmit() {
    if (!this.userForm.valid) {
      return false;
    } else {
      this.userCrudService.createUser(this.userForm.value)
        .subscribe((response) => {
          this.zone.run(() => {
            this.userForm.reset();
            this.router.navigate(['/list']);
          })
        });
    }
  }

}
```

You need to open the **create page TypeScript file**, here you need to add a **name, email, and username input fields** and set up the with the ngModel attribute.

Update **create.page.html** file:

```markup
<ion-header>
  <ion-toolbar>
    <ion-title>Create User</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content>
  <ion-list lines="full">
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <ion-item>
        <ion-label position="floating">Name</ion-label>
        <ion-input formControlName="name" type="text" required></ion-input>
      </ion-item>

      <ion-item>
        <ion-label position="floating">Email</ion-label>
        <ion-input formControlName="email" type="text" required>
        </ion-input>
      </ion-item>

      <ion-item>
        <ion-label position="floating">User name</ion-label>
        <ion-input formControlName="username" type="text" required>
        </ion-input>
      </ion-item>

      <ion-row>
        <ion-col>
          <ion-button type="submit" color="danger" expand="block">Add</ion-button>
        </ion-col>
      </ion-row>
    </form>
  </ion-list>
</ion-content>
```

## Ionic Http GET and Delete Example

We will show you how to send HTTP GET and Delete calls using HTTP; for making the HTTP call, we use the service.

Update **list.page.ts** file:

```typescript
import { Component, OnInit } from '@angular/core';
import { UserCrudService } from './../../services/user-crud.service';
import { Router } from '@angular/router';

@Component({
  selector: 'app-list',
  templateUrl: './list.page.html',
  styleUrls: ['./list.page.scss'],
})

export class ListPage implements OnInit {

  Users: any = [];

  constructor(
    private userCrudService: UserCrudService,
    private router: Router
  ) { }

  ngOnInit() {  }

  ionViewDidEnter() {
    this.userCrudService.getUsers().subscribe((response) => {
      this.Users = response;
    })
  }

  removeUser(user, i) {
    if (window.confirm('Are you sure')) {
      this.userCrudService.deleteUser(user.id)
        .subscribe(() => {
            this.ionViewDidEnter();
            console.log('User deleted!')
          }
        )
    }
  }

}

```

Open the HTML page, show the data in the ion-list item, hence place code in **list.page.html** file:

```markup
<ion-header>
  <ion-toolbar>
    <ion-title>Users List</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content>

  <ion-list>
    <ion-item *ngFor="let user of Users">
      <ion-label>
        <h2>{{user.name}}</h2>
        <h3>{{user.email}}</h3>
        <h3>{{user.username}}</h3>
      </ion-label>

      <div class="item-note" item-end>
        <button ion-button clear [routerLink]="['/update/', user.id]">
          <ion-icon name="create" style="zoom:1.5"></ion-icon>
        </button>
        <button ion-button clear (click)="removeUser(user)">
          <ion-icon name="trash" style="zoom:1.5"></ion-icon>
        </button>
      </div>
    </ion-item>
  </ion-list>

</ion-content>
```

## Ionic Http Put Example

Next, we will send HTTP Put Request in Ionic to update the data on the server, and we have defined the updateUser() method and access it via the angular service.

Open **update.module.ts** file and **import ReactiveFormsModule**.

```typescript
import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
  imports: [
    ReactiveFormsModule
  ]
})
```

Update **update.page.ts** file:

```typescript
import { Component, OnInit } from '@angular/core';

import { Router, ActivatedRoute } from "@angular/router";
import { FormGroup, FormBuilder } from "@angular/forms";
import { UserCrudService } from './../../services/user-crud.service';


@Component({
  selector: 'app-update',
  templateUrl: './update.page.html',
  styleUrls: ['./update.page.scss'],
})

export class UpdatePage implements OnInit {

  updateUserFg: FormGroup;
  id: any;

  constructor(
    private userCrudService: UserCrudService,
    private activatedRoute: ActivatedRoute,
    public formBuilder: FormBuilder,
    private router: Router
  ) {
    this.id = this.activatedRoute.snapshot.paramMap.get('id');
  }

  ngOnInit() {
    this.fetchUser(this.id);
    this.updateUserFg = this.formBuilder.group({
      name: [''],
      email: [''],
      username: ['']
    })
  }

  fetchUser(id) {
    this.userCrudService.getUser(id).subscribe((data) => {
      this.updateUserFg.setValue({
        name: data['name'],
        email: data['email'],
        username: data['username']
      });
    });
  }

  onSubmit() {
    if (!this.updateUserFg.valid) {
      return false;
    } else {
      this.userCrudService.updateUser(this.id, this.updateUserFg.value)
        .subscribe(() => {
          this.updateUserFg.reset();
          this.router.navigate(['/list']);
        })
    }
  }

}

```

In the update page, create a form with input values, so update **update.page.html** file:

```markup
<ion-header>
  <ion-toolbar>
    <ion-title>Update User</ion-title>
  </ion-toolbar>
</ion-header>


<ion-content>
  <ion-list lines="full">
    <form [formGroup]="updateUserFg" (ngSubmit)="onSubmit()">
      
      <ion-item>
        <ion-label position="floating">Name</ion-label>
        <ion-input formControlName="name" type="text" required></ion-input>
      </ion-item>

      <ion-item>
        <ion-label position="floating">Email</ion-label>
        <ion-input formControlName="email" type="text" required>
        </ion-input>
      </ion-item>

      <ion-item>
        <ion-label position="floating">User name</ion-label>
        <ion-input formControlName="username" type="text" required>
        </ion-input>
      </ion-item>      

      <ion-row>
        <ion-col>
          <ion-button type="submit" color="success" expand="block">Update</ion-button>
        </ion-col>
      </ion-row>
      
    </form>
  </ion-list>
</ion-content>
```

## Test Ionic App

To run the app on the browser, you may install the ionic lab package.

```bash
npm i @ionic/lab --save-dev
```

Initialize app on the browser:

```bash
ionic serve -l
```


## Conclusion

The Ionic HttpClient tutorial is over; we have explained how to communicate with the server using HTTP methods. We also explored how to set up a fake json server to consume REST APIs via GET, POST, PUT and delete methods.