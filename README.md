# Description
The project is a Signle Page Appliation, it uses react as frontend framework, Nest as backend framework. Both frontend and backend are using typescript. 
The project is using sqlite as backend database, for demo purpose, to eliminate database configuration.
It has the following features:

## Backend 
- JWT authorization and authentication
- Generaral Entity CRUD Web Api
- Type Orm as entity framework
## Frontend
- User Login and User Profile 
- User CRUD and Client CRUD
- Material UI Design

# Share code between frontend and backend
Authough both frontend and backend are written by typescript, it's tricky to put code in to same repository 
my ideal strutrue would be like
- -backend/src
- -frontend/src
- -shared/src
in this way, main.js of the backend need to refrence some file in ../../shared/src, it didn't work after I publish the project production enviroment. 
so share files must reside in backend src directory, so I changed the structure as following
## the code structure 
- package.json
- -src
- -[nest code]
- -src\client
- -scr\client\package.json
- -src\client\src
- -scr\client\src\share //entities, shared utility functions,etc...
- -src\client\src\[react code]

the root directory of the project is the backend code, it was generated by nest cli
the src\client direcotry is frontend code, it is a react app  generated by 'create react app'
both nest app and react app have their own package.json, so frontend and backend can has their own node_modules.

### some configurations
add the following line to src\client\.env, otherwise react app won't compile
```
SKIP_PREFLIGHT_CHECK=true
```
also set baseUrl to src in both package.json and src/client/package.json
```
    "baseUrl": "./src",
```
add src/client to tsconfig.build.json
```
"exclude": ["node_modules", "test", "dist","src/client", "**/*spec.ts"]
```
## run both frontend and backend in development mode
- start backend first 
```
yarn start:dev
```
- start the frontend
```
cd /src/clinet
yarn start
```
the headend is listening to port 4000, the frontend is listening to port 3000
add the following code to src/client/package.json, so if the frontend requests to a backend webapi, eg 'api/sign/sign', it would send the request to 'http://localhost:4000/api/sign/sign' 
```
  "proxy": "http://localhost:4000/",
```

## the file structure of the published
frontend and backend can be put to seperate app service, I prefered to put them in the same app service to save costs, also I don't need to enable cros.

After deploy to app service, the structure is like
- main.js
- client //here is all react code
- client/src/share  //the code shared between frontend and backend
- other headend code
- node_modules

### let nest to Serve Static
a client request was handled by nest router first, if nest router find the url match a controller, then it calls the web api, otherwise it looks if there is a static file in the client directory
```
 ServeStaticModule.forRoot({
      rootPath: join(__dirname, 'client')
    }),
```

## share the entity code
typeorm require add decorator @entity to class, and add @column decorator to class properies.
also I want to use a lib 'class-validator' to validate form inputs.
so the entity code ends up as the following
```
import { Entity, Unique,Column, ManyToOne } from 'decorators/myTypeOrmDecorator';
import { IsEmail, IsString, MinLength, MaxLength, Matches} from 'class-validator';
@Entity('user')
@Unique(['username'])
export class User extends MyBaseEntity {
  @IsEmail()
  @Column()
  username: string = '';

  @MinLength(1)
  @Column()
  firstname: string ='';
    ...
```
For the backend project, I can import @Entity(), @Column() from typeorm, but I don't want to do that for frontend project.
so instead of import Entity, Column from typeorm directly, I import the decorators from myTypeOrmDecorator
```
import { Entity, Unique,Column, ManyToOne } from 'decorators/myTypeOrmDecorator';
```
because in both package.json and src/client/package.json, I set baseUrl to 'src', 'decorators/myTypeOrmDecorator' refrence to diffrent files for frontend project and backend project
- for frontend project, it refrence to 'src/client/src/decorators/myTypeOrmDecorator'
- for backend project, it refrence to 'src/decorators/myTypeOrmDecorator'

in 'src/decorators/myTypeOrmDEcorator', I export * from typeorm
```
export * from "typeorm";
```
in 'src/client/src/decorators/myTypeOrmDecorator', I create some dummy decorators, the decorators do nothing, just to let frontend code be compiled


todo: when create use, if validate fail, it clear the form, maybe useMemory?
