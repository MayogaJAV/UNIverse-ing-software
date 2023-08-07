# LABORATORIO 09: Estilos de Programación

En este repositorio, encontrarás diferentes proyectos desarrollados en React y MongoDB, cada uno de ellos implementando un estilo de programación específico. A continuación, se describen los estilos utilizados en cada código:

# Monolithic Style

El estilo monolítico se refiere a la implementación de una aplicación o sistema en una única unidad lógica y cohesiva. En el código a continuación, el archivo `app.js` utiliza la arquitectura monolítica al conectar a MongoDB, definir rutas y crear el servidor express en una única aplicación.

### app.js

```javascript

import express from "express";
import cors from "cors";
import dotenv from "dotenv";
import connectDB from "./config/db.js";
import userRoute from "./routes/userRoute.js";

dotenv.config();

connectDB();

const app = express();

app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cors());

app.use("/api/users", userRoute);

app.get("/", (req, res) => {
  res.send("API is running...");
});

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

# Cook Book Style

El estilo `Cook Book` se enfoca en separar cada función por su funcionalidad específica y reutilizarlas en diferentes rutas o componentes. En el siguiente código, el controlador `UserController.js` muestra este estilo al definir funciones separadas para autenticar usuarios, registrar nuevos usuarios y obtener detalles de usuarios.

### UserController.js

```javascript
import asyncHandler from "express-async-handler";
import User from "../models/userModel.js";
import generateToken from "../utils/generateToken.js";

/*
 * @desc    Auth user & get token
 * @route   POST /api/users/login
 * @access  Public
 */
const authUser = asyncHandler(async (req, res) => {
  const { email, password } = req.body;

  const user = await User.findOne({ email });

  if (user && (await user.matchPassword(password))) {
    res.json({
      _id: user._id,
      username: user.username,
      firstname: user.firstname,
      lastname: user.lastname,
      email: user.email,
      isAdmin: user.isAdmin,
      token: generateToken(user._id),
    });
  } else {
    res.status(401);
    throw new Error("Invalid email or password");
  }
});

/*
 * @desc    Register a new user
 * @route   POST /api/users/
 * @access  Public
 */
const registerUser = asyncHandler(async (req, res) => {
  const { username, password, firstname, lastname, email } = req.body;

  const userExists = await User.findOne({ email });

  if (userExists) {
    res.status(400);
    throw new Error("User already exists");
  }

  const user = await User.create({
    username,
    password,
    firstname,
    lastname,
    email,
  });

  if (user) {
    res.status(201).json({
      _id: user._id,
      username: user.username,
      firstname: user.firstname,
      lastname: user.lastname,
      email: user.email,
      isAdmin: user.isAdmin,
      token: generateToken(user._id),
    });
  } else {
    res.status(400);
    throw new Error("Invalid user data");
  }
});

/*
 * @desc    Get user profile
 * @route   GET /api/users/profile
 * @access  Private
 */
const getUserProfile = asyncHandler(async (req, res) => {
  const user = await User.findById(req.user._id);

  if (user) {
    res.json({
      _id: user._id,
      username: user.username,
      firstname: user.firstname,
      lastname: user.lastname,
      email: user.email,
      isAdmin: user.isAdmin,
    });
  } else {
    res.status(404);
    throw new Error("User not found");
  }
});

/*
 * @desc    Update user profile
 * @route   PUT /api/users/profile
 * @access  Private
 */
const updateUserProfile = asyncHandler(async (req, res) => {
  const user = await User.findById(req.user._id);

  if (user) {
    user.username = req.body.username || user.username;
    user.firstname = req.body.firstname || user.firstname;
    user.lastname = req.body.lastname || user.lastname;
    user.email = req.body.email || user.email;
    if (req.body.password) {
      user.password = req.body.password;
    }

    const updatedUser = await user.save();

    res.json({
      _id: updatedUser._id,
      username: updatedUser.username,
      firstname: updatedUser.firstname,
      lastname: updatedUser.lastname,
      email: updatedUser.email,
      isAdmin: updatedUser.isAdmin,
      token: generateToken(updatedUser._id),
    });
  } else {
    res.status(404);
    throw new Error("User not found");
  }
});

/*
 * @desc    Get all users
 * @route   GET /api/users
 * @access Private/Admin
*/
const getUsers = asyncHandler(async (req, res) => {
const users = await User.find({});
res.json(users);
});
/*
@desc Delete user
@route DELETE /api/users/:id
@access Private/Admin
*/
const deleteUser = asyncHandler(async (req, res) => {
    const user = await User.findById(req.params.id);
    if (user) {
        await user.remove();
    res.json({ message: "User removed" });
    } else {
    res.status(404);
    throw new Error("User not found");
    }
    });
    
    /*
    @desc Get user by ID
    @route GET /api/users/:id
    @access Private/Admin
    */
    const getUserById = asyncHandler(async (req, res) => {
        const user = await User.findById(req.params.id).select("-password");
        if (user) {
        res.json(user);
        } else {
        res.status(404);
        throw new Error("User not found");
        }
        });
        
        /*
        @desc Update user
        @route PUT /api/users/:id
        @access Private/Admin
        */
const updateUser = asyncHandler(async (req, res) => {
    const user = await User.findById(req.params.id);
    if (user) {
        user.username = req.body.username || user.username;
        user.firstname = req.body.firstname || user.firstname;
        user.lastname = req.body.lastname || user.lastname;
        user.email = req.body.email || user.email;
        user.isAdmin = req.body.isAdmin || user.isAdmin;
        const updatedUser = await user.save();
        res.json({
            _id: updatedUser._id,
            username: updatedUser.username,
            firstname: updatedUser.firstname,
            lastname: updatedUser.lastname,
            email: updatedUser.email,
            isAdmin: updatedUser.isAdmin,
        });
        } else {
            res.status(404);
            throw new Error("User not found");
        }
    });
export { authUser, registerUser, getUserProfile, updateUserProfile, getUsers, deleteUser, getUserById, updateUser };
```
# Pipeline Style

El estilo `Pipeline` implica ejecutar múltiples funciones en un orden específico, encadenando el procesamiento para lograr un resultado deseado. En el siguiente código, el middleware `AuthMiddleware.js` implementa este estilo al ejecutar funciones de autenticación y autorización de usuarios en un orden secuencial.

### AuthMiddleware.js

```javascript
import jwt from "jsonwebtoken";
import asyncHandler from "express-async-handler";
import User from "../models/userModel.js";

const protect = asyncHandler(async (req, res, next) => {
  let token;

  if (req.headers.authorization && req.headers.authorization.startsWith("Bearer")) {
    try {
      token = req.headers.authorization.split(" ")[1];

      const decoded = jwt.verify(token, process.env.JWT_SECRET);

      req.user = await User.findById(decoded.id).select("-password");

      next();
    } catch (error) {
      console.error(error);
      res.status(401);
      throw new Error("Not authorized, token failed");
    }
  }

  if (!token) {
    res.status(401);
    throw new Error("Not authorized, no token");
  }
});

const admin = (req, res, next) => {
  if (req.user && req.user.isAdmin) {
    next();
  } else {
    res.status(401);
    throw new Error("Not authorized as an admin");
  }
};

export { protect, admin };
```

# Thing Style

El estilo `Thing` se refiere a la definición de estructura y comportamiento de una entidad en una clase o módulo separado. En el siguiente código, el modelo de datos `UserModel.js` muestra este estilo al definir la estructura y comportamiento de la entidad de usuario en una clase.

### UserModel.js

```javascript
import mongoose from "mongoose";

const userSchema = new mongoose.Schema(
  {
    username: { type: String, required: true, unique: true },
    password: { type: String, required: true },
    firstname: { type: String, required: true },
   lastname: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    isAdmin: { type: Boolean, required: true, default: false },
  },
  {
    timestamps: true,
  }
);

const User = mongoose.model("User", userSchema);

export default User;
```
# Persistent Tables Style
El estilo `Persistent Tables` se refiere a la implementación de rutas y componentes en React que muestran tablas o elementos persistentes en la interfaz. En el siguiente código, el archivo `app.js` implementa este estilo al definir rutas de React y renderizar tablas de usuarios y otros elementos persistentes en la interfaz.

### app.js

```javascript
import express from "express";
import cors from "cors";
import dotenv from "dotenv";
import connectDB from "./config/db.js";
import userRoute from "./routes/userRoute.js";

dotenv.config();

connectDB();

const app = express();

app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cors());

app.use("/api/users", userRoute);

app.get("/", (req, res) => {
  res.send("API is running...");
});

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

# Map-Reduce Style
El estilo `Map-Reduce` implica el uso de funciones de mapeo y reducción para procesar datos o generar resultados. En el siguiente código, el archivo `generateToken.js` sigue este estilo al utilizar una función de mapeo para generar tokens de autenticación.

### generateToken.js

```javascript
import jwt from "jsonwebtoken";

const generateToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_SECRET, {
    expiresIn: "30d",
  });
};

export default generateToken;
```

# LABORATORIO 10: Codificación Legible (Clean Code)

Clean Code, o "Código Limpio", es una filosofía y conjunto de prácticas de programación que busca producir software legible, fácil de entender, bien estructurado y mantenible. La teoría de Clean Code se basa en la idea de que el código es leído muchas más veces de lo que es escrito, y que el tiempo y esfuerzo dedicados a escribir un código limpio y claro valen la pena a largo plazo, ya que facilitan su mantenimiento y reducen la probabilidad de errores y fallos.

## userController.js

* Nombres significativos: Las variables y funciones tienen nombres descriptivos que indican claramente su propósito y función. Por ejemplo, authUser indica que esta función se utiliza para autenticar al usuario.
* Funciones cortas y concisas: La función authUser es concisa y se enfoca en una tarea específica, lo que facilita su comprensión y mantenimiento.
* Comentarios significativos: Se utilizan comentarios para describir el propósito de la función y la ruta de acceso a la que corresponde.
* Separación de responsabilidades: La función authUser maneja la autenticación del usuario y la generación del token, lo que contribuye a una mejor organización del código.

```javascript
import asyncHandler from "express-async-handler";
import User from "../models/userModel.js";
import generateToken from "../utils/generateToken.js";

/*
 * @desc    Auth user & get token
 * @route   POST /api/users/login
 * @access  Public
 */
const authUser = asyncHandler(async (req, res) => {
  const { email, password } = req.body;

  const user = await User.findOne({ email });

  if (user && (await user.matchPassword(password))) {
    res.json({
      _id: user._id,
      username: user.username,
      firstname: user.firstname,
      lastname: user.lastname,
      email: user.email,
      isAdmin: user.isAdmin,
      token: generateToken(user._id),
    });
  } else {
    res.status(401);
    throw new Error("Invalid email or password");
  }
});

/*
 * @desc    Register a new user
 * @route   POST /api/users/
 * @access  Public
 */
const registerUser = asyncHandler(async (req, res) => {
  const { username, password, firstname, lastname, email } = req.body;

  const userExists = await User.findOne({ email });

  if (userExists) {
    res.status(400);
    throw new Error("User already exists");
  }

  const user = await User.create({
    username,
    password,
    firstname,
    lastname,
    email,
  });

  if (user) {
    res.status(201).json({
      _id: user._id,
      username: user.username,
      firstname: user.firstname,
      lastname: user.lastname,
      email: user.email,
      isAdmin: user.isAdmin,
      token: generateToken(user._id),
    });
  } else {
    res.status(400);
    throw new Error("Invalid user data");
  }
});

/*
 * @desc    Get user profile
 * @route   GET /api/users/profile
 * @access  Private
 */
const getUserProfile = asyncHandler(async (req, res) => {
  const user = await User.findById(req.user._id);

  if (user) {
    res.json({
      _id: user._id,
      username: user.username,
      firstname: user.firstname,
      lastname: user.lastname,
      email: user.email,
      isAdmin: user.isAdmin,
    });
  } else {
    res.status(404);
    throw new Error("User not found");
  }
});

/*
 * @desc    Update user profile
 * @route   PUT /api/users/profile
 * @access  Private
 */
const updateUserProfile = asyncHandler(async (req, res) => {
  const user = await User.findById(req.user._id);

  if (user) {
    user.username = req.body.username || user.username;
    user.firstname = req.body.firstname || user.firstname;
    user.lastname = req.body.lastname || user.lastname;
    user.email = req.body.email || user.email;
    if (req.body.password) {
      user.password = req.body.password;
    }

    const updatedUser = await user.save();

    res.json({
      _id: updatedUser._id,
      username: updatedUser.username,
      firstname: updatedUser.firstname,
      lastname: updatedUser.lastname,
      email: updatedUser.email,
      isAdmin: updatedUser.isAdmin,
      token: generateToken(updatedUser._id),
    });
  } else {
    res.status(404);
    throw new Error("User not found");
  }
});

/*
 * @desc    Get all users
 * @route   GET /api/users
 * @access Private/Admin
*/
const getUsers = asyncHandler(async (req, res) => {
const users = await User.find({});
res.json(users);
});
/*
@desc Delete user
@route DELETE /api/users/:id
@access Private/Admin
*/
const deleteUser = asyncHandler(async (req, res) => {
    const user = await User.findById(req.params.id);
    if (user) {
        await user.remove();
    res.json({ message: "User removed" });
    } else {
    res.status(404);
    throw new Error("User not found");
    }
    });
    
    /*
    @desc Get user by ID
    @route GET /api/users/:id
    @access Private/Admin
    */
    const getUserById = asyncHandler(async (req, res) => {
        const user = await User.findById(req.params.id).select("-password");
        if (user) {
        res.json(user);
        } else {
        res.status(404);
        throw new Error("User not found");
        }
        });
        
        /*
        @desc Update user
        @route PUT /api/users/:id
        @access Private/Admin
        */
const updateUser = asyncHandler(async (req, res) => {
    const user = await User.findById(req.params.id);
    if (user) {
        user.username = req.body.username || user.username;
        user.firstname = req.body.firstname || user.firstname;
        user.lastname = req.body.lastname || user.lastname;
        user.email = req.body.email || user.email;
        user.isAdmin = req.body.isAdmin || user.isAdmin;
        const updatedUser = await user.save();
        res.json({
            _id: updatedUser._id,
            username: updatedUser.username,
            firstname: updatedUser.firstname,
            lastname: updatedUser.lastname,
            email: updatedUser.email,
            isAdmin: updatedUser.isAdmin,
        });
        } else {
            res.status(404);
            throw new Error("User not found");
        }
    });
export { authUser, registerUser, getUserProfile, updateUserProfile, getUsers, deleteUser, getUserById, updateUser };
```

## authMiddleware.js

* Nombres significativos: Las variables y funciones tienen nombres descriptivos que indican claramente su propósito. Por ejemplo, protect y admin indican sus funciones de protección y administración respectivamente.
* Funciones cortas y concisas: Las funciones protect y admin son concisas y se enfocan en tareas específicas, lo que facilita su comprensión y mantenimiento.
* Separación de responsabilidades: La función protect maneja la verificación del token de autorización, mientras que admin verifica si el usuario tiene permisos de administrador, lo que contribuye a una mejor organización del código.

```javascript
import jwt from "jsonwebtoken";
import asyncHandler from "express-async-handler";
import User from "../models/userModel.js";

const protect = asyncHandler(async (req, res, next) => {
  let token;

  if (req.headers.authorization && req.headers.authorization.startsWith("Bearer")) {
    try {
      token = req.headers.authorization.split(" ")[1];

      const decoded = jwt.verify(token, process.env.JWT_SECRET);

      req.user = await User.findById(decoded.id).select("-password");

      next();
    } catch (error) {
      console.error(error);
      res.status(401);
      throw new Error("Not authorized, token failed");
    }
  }

  if (!token) {
    res.status(401);
    throw new Error("Not authorized, no token");
  }
});

const admin = (req, res, next) => {
  if (req.user && req.user.isAdmin) {
    next();
  } else {
    res.status(401);
    throw new Error("Not authorized as an admin");
  }
};

export { protect, admin };
```

## userModel.js
* Nombres significativos: Los nombres de los campos del esquema tienen significados claros, lo que facilita la comprensión de la estructura del modelo de usuario.

```javascript
import mongoose from "mongoose";

const userSchema = new mongoose.Schema(
  {
    username: { type: String, required: true, unique: true },
    password: { type: String, required: true },
    firstname: { type: String, required: true },
   lastname: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    isAdmin: { type: Boolean, required: true, default: false },
  },
  {
    timestamps: true,
  }
);

const User = mongoose.model("User", userSchema);

export default User;
```

## userRoute.js
* Nombres significativos: Los nombres de las rutas y funciones son descriptivos y fáciles de entender. Por ejemplo, authUser, registerUser, getUserProfile, etc., indican sus respectivas funciones.
* Funciones cortas y concisas: Cada función en el archivo se enfoca en una tarea específica, lo que facilita la lectura y el mantenimiento del código.
* Comentarios significativos: Se utilizan comentarios para describir el propósito de cada ruta y función.

```javascript
import jwt from "jsonwebtoken";

const generateToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_SECRET, {
    expiresIn: "30d",
  });
};

export default generateToken;
```

## generateToken.js

* Nombres significativos: El nombre generateToken indica claramente que esta función se utiliza para generar un token de autenticación.

```javascript
import jwt from "jsonwebtoken";

const generateToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_SECRET, {
    expiresIn: "30d",
  });
};

export default generateToken;
```

## app.js

* Nombres significativos: Los nombres de las variables y funciones son descriptivos y reflejan su propósito. Por ejemplo, app, userRoute, PORT, etc.
* Funciones cortas y concisas: La función principal del servidor es concisa y se centra en la configuración y el enrutamiento del servidor.
* Separación de responsabilidades: Las rutas se manejan mediante el uso de userRoute, lo que ayuda a mantener una estructura organizada del código.

```javascript
import express from "express";
import cors from "cors";
import dotenv from "dotenv";
import connectDB from "./config/db.js";
import userRoute from "./routes/userRoute.js";

dotenv.config();

connectDB();

const app = express();

app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cors());

app.use("/api/users", userRoute);

app.get("/", (req, res) => {
  res.send("API is running...");
});

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

# LABORATORIO 11: Aplicación de Principios SOLID en el Código
Los `Principios SOLID` son un conjunto de cinco directrices de diseño de software que buscan mejorar la mantenibilidad, extensibilidad y comprensión del código. Estos principios fueron propuestos por Robert C. Martin y son una referencia fundamental para escribir código limpio y de calidad. A continuación, se describen brevemente cada uno de los principios:

* Principio de Responsabilidad Única (SRP - Single Responsibility Principle): 
Este principio establece que una clase o módulo debe tener una única razón para cambiar. En otras palabras, una clase debe tener una única responsabilidad. Esto mejora la cohesión y facilita el mantenimiento del código al reducir la probabilidad de efectos secundarios no deseados.

* Principio de Abierto/Cerrado (OCP - Open/Closed Principle): 
El principio OCP dicta que una entidad de software (clase, módulo, función) debe estar abierta para la extensión pero cerrada para la modificación. En lugar de modificar el código existente, se deben agregar nuevas funcionalidades mediante extensiones.

* Principio de Sustitución de Liskov (LSP - Liskov Substitution Principle): 
Este principio establece que los objetos de una clase derivada deben poder sustituirse por objetos de la clase base sin afectar la integridad del programa. En otras palabras, una subclase debe ser reemplazable por su clase base sin cambiar el comportamiento esperado.

* Principio de Segregación de la Interfaz (ISP - Interface Segregation Principle): 
El ISP propone que una interfaz no debe forzar a las clases que la implementan a depender de métodos que no utilizan. En lugar de tener interfaces genéricas, se deben crear interfaces específicas para cada contexto.

* Principio de Inversión de Dependencia (DIP - Dependency Inversion Principle): 
El DIP establece que los módulos de alto nivel no deben depender de módulos de bajo nivel. Ambos deben depender de abstracciones. Además, las abstracciones no deben depender de los detalles; los detalles deben depender de las abstracciones.

Estos principios ayudan a crear código más limpio, modular y fácil de mantener. A continuación, se detalla cómo se aplican algunos de estos principios en los códigos proporcionados:

# Aplicación de Principios SOLID en el Código
## userModel.js (Definición de esquema de usuario)
* Principio de Responsabilidad Única (SRP): El archivo userModel.js cumple con el principio SRP al definir exclusivamente la estructura del modelo de usuario, separando así la responsabilidad de la definición del esquema del resto de la aplicación.
## userController.js (Controladores para operaciones de usuario)
* Principio de Responsabilidad Única (SRP): Cada controlador en el archivo userController.js sigue el principio SRP al encargarse de una única operación relacionada con el usuario. Por ejemplo, authUser se encarga de la autenticación de usuarios, registerUser de la creación de nuevos usuarios, y así sucesivamente.
* Principio de Abierto/Cerrado (OCP): Los controladores en este archivo cumplen con el principio OCP al permitir la extensión para agregar nuevas operaciones sin modificar el código existente. Esto se logra mediante la adición de nuevos controladores y rutas sin afectar los existentes.
## authMiddleware.js (Middlewares de autenticación)
* Principio de Responsabilidad Única (SRP): El middleware de autenticación en authMiddleware.js cumple con el principio SRP al enfocarse únicamente en la verificación de autenticación a través de tokens.
* Principio de Sustitución de Liskov (LSP): Aunque no es evidente en el código proporcionado, el principio LSP se aplica si el middleware de autenticación puede sustituirse por otro middleware sin alterar el comportamiento esperado.
## generateToken.js (Generación de tokens JWT)
* Principio de Responsabilidad Única (SRP): La función generateToken en generateToken.js sigue el principio SRP al tener la única responsabilidad de generar tokens JWT.
## userRoute.js (Rutas para las operaciones de usuario)
* Principio de Responsabilidad Única (SRP): Cada ruta y su controlador correspondiente en userRoute.js aplican el principio SRP al manejar una única operación relacionada con los usuarios, como autenticación, registro, obtención de perfiles, entre otras.
* Principio de Abierto/Cerrado (OCP): Las rutas y controladores en este archivo cumplen con el principio OCP al permitir la extensión para agregar nuevas operaciones sin modificar el código existente.
## index.js (Configuración y ejecución del servidor)
* Principio de Responsabilidad Única (SRP): El archivo index.js sigue el principio SRP al ser responsable de la configuración y ejecución del servidor, manteniendo así una única responsabilidad.
