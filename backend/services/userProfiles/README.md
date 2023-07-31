# LABORATORIO 09: Estilos de Programación

En este repositorio, encontrarás diferentes proyectos desarrollados en React y MongoDB, cada uno de ellos implementando un estilo de programación específico. A continuación, se describen los estilos utilizados en cada código:

# Monolithic Style

El estilo monolítico se refiere a la implementación de una aplicación o sistema en una única unidad lógica y cohesiva. En el código a continuación, el archivo `index.js` utiliza la arquitectura monolítica al conectar a MongoDB, definir rutas y crear el servidor express en una única aplicación.

### index.js

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
