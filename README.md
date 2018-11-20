# loopback-custom-roles

## Loopback custom roles or security

To use a custom role you can created a custom tables to manage this:

In my case I use this tables:

* usuario - (Table of users) 
* usuario-grupos - (Table of users asociate a any groups) 
* grupos - (It contains any models to acces) 
* modelos - (Models who can acces by a group)
* permisos - (Permisions for models)


### Example of data in json of roles for one user:

```` json
[
    {
        "realm": null,
        "username": "david",
        "email": "david@doe.com",
        "emailVerified": null,
        "id": 3,
        "usuario_grupos": [
            {
                "id": 1,
                "id_usuario": 3,
                "id_grupo": 1,
                "grupos": [
                    {
                        "id": 1,
                        "nombre": "GRUPO_MOVIMIENTOS_ADMIN",
                        "descripcion": null,
                        "modelos": [
                            {
                                "id": 1,
                                "nombre": "movimientos",
                                "id_grupo": 1,
                                "permisos": [
                                    {
                                        "id": 1,
                                        "nombre": "read",
                                        "id_modelo": 1
                                    },
                                    {
                                        "id": 2,
                                        "nombre": "write",
                                        "id_modelo": 1
                                    },
                                    {
                                        "id": 3,
                                        "nombre": "update",
                                        "id_modelo": 1
                                    },
                                    {
                                        "id": 4,
                                        "nombre": "delete",
                                        "id_modelo": 1
                                    }
                                ]
                            },
                            {
                                "id": 2,
                                "nombre": "cementerio",
                                "id_grupo": 1,
                                "permisos": [
                                    {
                                        "id": 10,
                                        "nombre": "read",
                                        "id_modelo": 2
                                    },
                                    {
                                        "id": 11,
                                        "nombre": "write",
                                        "id_modelo": 2
                                    },
                                    {
                                        "id": 12,
                                        "nombre": "update",
                                        "id_modelo": 2
                                    },
                                    {
                                        "id": 13,
                                        "nombre": "delete",
                                        "id_modelo": 2
                                    }
                                ]
                            },
                            {
                                "id": 3,
                                "nombre": "difunto",
                                "id_grupo": 1,
                                "permisos": [
                                    {
                                        "id": 14,
                                        "nombre": "read",
                                        "id_modelo": 3
                                    },
                                    {
                                        "id": 15,
                                        "nombre": "write",
                                        "id_modelo": 3
                                    },
                                    {
                                        "id": 16,
                                        "nombre": "update",
                                        "id_modelo": 3
                                    },
                                    {
                                        "id": 17,
                                        "nombre": "delete",
                                        "id_modelo": 3
                                    }
                                ]
                            },
                            {
                                "id": 4,
                                "nombre": "personas",
                                "id_grupo": 1,
                                "permisos": [
                                    {
                                        "id": 18,
                                        "nombre": "read",
                                        "id_modelo": 4
                                    },
                                    {
                                        "id": 19,
                                        "nombre": "write",
                                        "id_modelo": 4
                                    },
                                    {
                                        "id": 20,
                                        "nombre": "update",
                                        "id_modelo": 4
                                    },
                                    {
                                        "id": 21,
                                        "nombre": "delete",
                                        "id_modelo": 4
                                    }
                                ]
                            }
                        ]
                    }
                ]
            }
        ]
    }
]
````


### Data for restirct the acces in one model:
```` javascript
'use strict';

module.exports = function (Personas) {

    var app = require('../../server/server');
    var loopback = require('loopback');

    Personas.beforeRemote("*", function (context, unused, next) {

        console.log(context);

        var tipo = context.req.method;
        var metodo = context.methodString;

        // Obtenemos el usuario actual
        var userId = context.args.options.accessToken.userId;

        // Modelo de usuarios
        var Usuarios = app.models.Usuarios;

        var data = JSON.parse('{"where":{"id":' + userId + '},"include":{"usuario_grupos":{"grupos":{"modelos":"permisos"}}}}');

        Usuarios.find(data, function (err, user) {

            if (err) {
                console.log("err --> ", err);
                next(new Error(err));

            } else {
                console.log("user --> ", user);
                var usuario = user[0].toJSON();

                var arrayDatosfinales = [];
                let noNeg = usuario.usuario_grupos.filter(function (index) {
                    index.grupos.filter(function (index) {
                        index.modelos.filter(function (index) {
                            var json = {
                                "nombre": index.nombre,
                                "permisos": index.permisos,
                            }
                            arrayDatosfinales[index.nombre] = json;
                            console.log(" --- " + JSON.stringify(index));
                        })
                    })
                });

                console.log(arrayDatosfinales);

                try {
                    var compararPermiso = "";

                    if (tipo === "GET") {
                        compararPermiso = "read";
                    } else if (tipo === "POST") {
                        compararPermiso = "create";
                    } else if (tipo === "PUT") {
                        compararPermiso = "update";
                    } else if (tipo === "DELETE") {
                        compararPermiso = "delete";
                    }

                    // Comprobamos si tenemos algun permiso
                    var permiso = arrayDatosfinales['movimientos'].permisos.filter(function (index) {
                        return index.nombre == compararPermiso;
                    });
                    console.log(permiso);

                    if (permiso.length > 0) {
                        next();
                    } else {
                        // without permisions
                        var error = new Error("No dispones de los permisos suficientes");
                        context.res.statusCode = 403;
                        next(error);
                    }

                } catch (error) {
                    // without permisions
                    var error = new Error("No dispones de los permisos suficientes");
                    context.res.statusCode = 403;
                    next(error);
                }
            }
        });
    });

};
````
