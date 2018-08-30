# ORM 

### idea
Construcción de un ORM con modelos, migraciones y soporte para diferentes motores de base de datos.



### Plantemianiento del problema
Actualmente existen varios ORM, de los cuales entre los mas populares destacan [sequelizejs](sequelizejs.com), pero la idea es realizar algo mas sencillo para su uso. Usando una forma de uso similar a [Lucid Models](https://adonisjs.com/docs/4.1/lucid) (desarrollado por  por AdonisJS) o [eloquent ORM](https://laravel.com/docs/5.0/eloquent) (usado por Laravel).


### Antecedentes
Algunos de los ORM que estan disponibles para NodeJS actualmente son los siguientes:
* ##### Sequelize
Posee soporte para PostgreSQL, MariaDB, MySQL, MS SQL. Un ejemplo de squelizer:
``````js
var Sequelize = require('sequelize');
var sequelize = new Sequelize('database', 'username', 'password');
 
var User = sequelize.define('user', {
  username: Sequelize.STRING,
  birthday: Sequelize.DATE
});
 
sequelize.sync().then(function() {
  return User.create({
    username: 'janedoe',
    birthday: new Date(1980, 6, 20)
  });
}).then(function(jane) {
  console.log(jane.get({
    plain: true
  }));
});
``````
* ##### node-orm2
ORM con soporte para MySQL, PostgreSQL, Amazon Redshift y SQLite. Un ejemplo de la node-orm2:
``````js
var orm = require("orm");
 
orm.connect("mysql://username:password@host/database", function (err, db) {
  if (err) throw err;
 
    var Person = db.define("person", {
        name      : String,
        surname   : String,
        age       : Number,
        male      : Boolean,
        continent : [ "Europe", "America", "Asia", "Africa", "Australia", "Antartica" ], // ENUM type
        photo     : Buffer, // BLOB/BINARY
        data      : Object // JSON encoded
    }, {
        methods: {
            fullName: function () {
                return this.name + ' ' + this.surname;
            }
        },
        validations: {
            age: orm.validators.rangeNumber(18, undefined, "under-age")
        }
    });
 
    Person.find({ surname: "Doe" }, function (err, people) {
        // SQL: "SELECT * FROM person WHERE surname = 'Doe'"
 
        console.log("People found: %d", people.length);
        console.log("First person: %s, age %d", people[0].fullName(), people[0].age);
 
        people[0].age = 16;
        people[0].save(function (err) {
            // err.msg = "under-age";
        });
    });
});
``````

Como estos ejemplo hay muchos más, pero su uso y estructura es muy similar.

#### ¿Porqué desarrollar está idea?

Mejoraria con respecto a tiempos y curva de aprendizaje con respecto a los disponibles actualmente, ya que su implementaci´ón seria más simple.

#### Beneficios

¡¡¡ Honor y gloria !!! 

además que puede ser una herramienta de mucha ayuda sobre todo para los que estan empezando, esta idea la empece a desarrollar ya que en estoy construyendo en mis tiempos libres una app en electron, y pues no encontre un ORM sencillo de usar, como los que venia acostumbrado en los frameworks de back. La idea es que sea simple su uso e implementación automatizando la construcción de las consultas de tal manera que de forma agíl se cubra este aspecto en futuros desarrollos.


#### Stack tecnológico

Las bases de datos que deberia soportar seria MySQL, MariaDB, PostgreSQL, MS SQL, MongoDB.

La idea es usar los drivers para nodeJS de estos motores y construir un ModelBase, teniendo en cuenta la estructura, restricciones y demás aspectos que contempla cada motor.


#### Progreso actual de la idea

Actualemente estoy trabajando para MySQL, puedo consultar la base de datos y crear modelos, manejo realaciones Muchos a Uno, y Uno a Uno, No he encontrado la forma de realizar el Muchos a muchos, ya que e ideado una forma de usar la tabla pibote para esto.

##### Avances

* ######  Configuración 
Se define una variable global en el index.js con la se definen los parametros de conexión, la idea es cambiar esto por un archivo .env
`````js
/*index.js file*/
var config = {
    db :{
        host     : 'localhost',
        user     : 'root',
        password : '1234',
        database : 'db_name'
    }
}
//nodejs globals vars
global.config = config;
``````

* ######  Operaciones directas a la base de datos (sin modelos)
Se puede realizar consultas, actualizaciones o eliminación directamente a la base de datos sin realizar uso de los modelos.

Ejemplo de select:
`````js
var DB = require('netsaj/mysql-client')
var db = new DB();

// SELECT ONE

var user = await db.table('users')
.where('email', '=','asd@domain.com')
.first();

// SELECT ALL
var users_list = await db.table('users')
.all();
`````

Para seleccionar campos especificos se realiza de la siguiente manera:
``````js
var users_list = await db.table('users')
.select('name, email')
.all();
``````

Ejemplo de condiciones where
`````js
var users_list = await db.table('users')
.where('active', '=', true)
.where('age', '>', 25)
.whereNull('age')
.select('name, email')
.all();
``````

Ejemplo de insert:
`````js
// the json var
var user = {
    name: 'fabio moreno',
    email: 'asd@domain.com',
    age: 25,
    active: true
}
  
// select the table and sent the json to create
cont id = await db.table('users')
.insert(user);
   
    
// return the autoincrement id
console.log(id)
print>_ 10
``````

Ejemplo de editar:
`````js
// the json var
var user = {
    name: 'fabio moreno',
    email: 'das@domain.com',
    age: 30,
    active: false
}
  
// select the table and sent the json to edit
cont afect = await db.table('users')
.where('id', '=', 10)
.edit(user);

// return the numbers of rows that afected by the query
console.log(afect)
print>_ 1

//or

cont afect = await db.table('users')
.set('email', 'das@domain.com')
.set('age',30)
.set('active',false)
.where('id', '=',10)
.update()
   
// return the numbers of rows that afected by the query
console.log(afect)
print>_ 1
``````
Ejemplo delete:
`````js
cont afect = await db.table('users')
.where('email', 'like', 'das@%')
.delete()
  
// return the numbers of rows that afected by the query
console.log(afect)
print>_ 1
``````

* ######  Usando modelos

Para este ejemplo usaremos dos modelos Roles y Usuarios. En el cual un Usuario tendra muchos roles. y pues como aún no he hecho el muchos a muchos toca así xD 

`````js
'use strict'
import BaseModel from "./BaseModel";
import Usuario from "./Usuario"

class Roles extends BaseModel{
    //campos de la tabla
    id;
    nombre;
    user_id;
    
    async usuario(){
        return await super.belongsTo(Usuario,"id","user_id");
    }
}
export default Roles;
``````
`````js
/** Usuario.js file **/
'use strict'
import BaseModel from "./BaseModel"; // modelo base
import Roles from "./Roles"; //Este es otro modelo al cual voy a hacer realación
class Usuario extends BaseModel{

    id;
    nombre;
    email;
    clave;
    //este metodo es cuando la tabla se llama diferente al archivo.
    table(){
        return 'users';
    }
    async roles(){
        return await super.hasMany(Roles, "id", "user_id"  );
    }
}



export default Usuario;
``````

Para usar los modelos se hace de la siguiente manera: 

`````js
import Usuario from '../models/Usuario'

var usr = new Usuario();
usr.email = "user@domain.com";
usr.password = Math.random()*1000;
await usr.save();

//como Usuario tiene un campo id autoincremento, al guardarlo usr.id se guarda el valor de este. y se podria usarse de una véz de requerise
console.log(user.id) //print id asigned.
``````

Una consulta se puede realizar de la siguiente manera:
`````js
var lista = await new Usuario().all();
console.log(lista); // esto me deolveria todos los registros

//tambien puedo hacer uso de los wheres descritos arriba 
var usr = await new Usuario().where('email', email).first();

// las relaciones se pueden consultar de una vez usando with o se realizan a medida que se necesiten solamente llamando el metodo conrrespondiente, un ejemplo de esto seria:

var lista = await new Usuario().with('roles,roles.usuario').all();
console.log(lista);
``````
Esto me traeria para cada Usuario, la realación 'roles' y le digo que tambien quiero que para cada uno de los elementos de la relacion roles, me traiga su usuario correspondiente. ( es un ejemplo absurdo pero es un ejemplo xD ) 

### Conclusiones

No llevo mucho de la idea global que tengo y voy a paso de tortuga pero, creo que seria una herramienta muy buena, siempre lo logre el objetivo que sea reducir el codigo para el usuario final.

Habria que al final desarrollar un CLI para que haga un mapeo de la base de datos y genere los modelos, reduciendo aún mas la escritura a pulso. 





