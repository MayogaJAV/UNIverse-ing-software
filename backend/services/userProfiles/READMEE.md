## Análisis de Estilos de Programación - UNIverse

### Componente `LogoSearch`

```jsx
import React from "react";
import Logo from "../../img/logo.png";
import './LogoSearch.css'
import {UilSearch} from '@iconscout/react-unicons';
import {BiSolidSearchAlt2} from "react-icons/bi";
import {DiJqueryUiLogo} from "react-icons/di";

const LogoSearch = () => {
    return (
        <div className="LogoSearch">
            <DiJqueryUiLogo className="unsalogoPrincipal"/>
            {/* <img src={Logo} alt=""/> */}
            <div className="Search">
                <input type="text" placeholder="#Explore"/>
                <div className="">
                    {/* <UilSearch/> */}
                    <BiSolidSearchAlt2 className="unsa"/>
                </div>
            </div>
        </div>
    );
};

export default LogoSearch;
```


**Cookbook (Recetario):** Se utiliza el componente `DiJqueryUiLogo` y `BiSolidSearchAlt2` de bibliotecas de iconos como "recetas" predefinidas para incorporar iconos en la interfaz de usuario.

**Abstract Things (Cosas Abstractas):** Utiliza componentes abstractos para representar elementos visuales, como iconos.

**Code Golf (Golf de código):** El código es relativamente corto, pero no parece optimizado para minimizar su longitud al máximo.

**The One (El Elegido):** El componente `LogoSearch` podría considerarse el "Elegido" para representar la parte de la interfaz de usuario que muestra el logotipo y la barra de búsqueda.

---

### Componente `ChatBox`

```jsx
import React, { useEffect, useState, useRef } from "react";
import { addMessage, getMessages } from "../../api/MessageRequests";
import { getUser } from "../../api/UserRequests";
import "./ChatBox.css";
import { format } from "timeago.js";
import InputEmoji from 'react-input-emoji'

const ChatBox = ({ chat, currentUser, setSendMessage,  receivedMessage }) => {
  const [userData, setUserData] = useState(null);
  const [messages, setMessages] = useState([]);
  const [newMessage, setNewMessage] = useState("");

  const handleChange = (newMessage)=> {
    setNewMessage(newMessage)
  }

  // fetching data for header
  useEffect(() => {
    const userId = chat?.members?.find((id) => id !== currentUser);
    const getUserData = async () => {
      try {
        const { data } = await getUser(userId);
        setUserData(data);
      } catch (error) {
        console.log(error);
      }
    };

    if (chat !== null) getUserData();
  }, [chat, currentUser]);

  // fetch messages
  useEffect(() => {
    const fetchMessages = async () => {
      try {
        const { data } = await getMessages(chat._id);
        setMessages(data);
      } catch (error) {
        console.log(error);
      }
    };

    if (chat !== null) fetchMessages();
  }, [chat]);

  // Always scroll to last Message
  useEffect(()=> {
    scroll.current?.scrollIntoView({ behavior: "smooth" });
  },[messages])

  // Send Message
  const handleSend = async(e)=> {
    e.preventDefault()
    const message = {
      senderId : currentUser,
      text: newMessage,
      chatId: chat._id,
    }
    const receiverId = chat.members.find((id)=>id!==currentUser);
    // send message to socket server
    setSendMessage({...message, receiverId})
    // send message to database
    try {
      const { data } = await addMessage(message);
      setMessages([...messages, data]);
      setNewMessage("");
    }
    catch
    {
      console.log("error")
    }
  }

  // Receive Message from parent component
  useEffect(()=> {
    console.log("Message Arrived: ", receivedMessage)
    if (receivedMessage !== null && receivedMessage.chatId === chat._id) {
      setMessages([...messages, receivedMessage]);
    }
  },[receivedMessage])

  const scroll = useRef();
  const imageRef = useRef();
  return (
    <>
      <div className="ChatBox-container">
        {chat ? (
          <>
            {/* chat-header */}
            <div className="chat-header">
              <div className="follower">
                <div>
                  <img
                    src={
                      userData?.profilePicture
                        ? process.env.REACT_APP_PUBLIC_FOLDER +
                          userData.profilePicture
                        : process.env.REACT_APP_PUBLIC_FOLDER +
                          "defaultProfile.png"
                    }
                    alt="Profile"
                    className="followerImage"
                    style={{ width: "50px", height: "50px" }}
                  />
                  <div className="name" style={{ fontSize: "0.9rem" }}>
                    <span>
                      {userData?.firstname} {userData?.lastname}
                    </span>
                  </div>
                </div>
              </div>
              <hr
                style={{
                  width: "95%",
                  border: "0

.1px solid #ececec",
                  marginTop: "20px",
                }}
              />
            </div>
            {/* chat-body */}
            <div className="chat-body" >
              {messages.map((message) => (
                <>
                  <div ref={scroll}
                    className={
                      message.senderId === currentUser
                        ? "message own"
                        : "message"
                    }
                  >
                    <span>{message.text}</span>{" "}
                    <span>{format(message.createdAt)}</span>
                  </div>
                </>
              ))}
            </div>
            {/* chat-sender */}
            <div className="chat-sender">
              <div onClick={() => imageRef.current.click()}>+</div>
              <InputEmoji
                value={newMessage}
                onChange={handleChange}
              />
              <div className="send-button button" onClick = {handleSend}>Send</div>
              <input
                type="file"
                name=""
                id=""
                style={{ display: "none" }}
                ref={imageRef}
              />
            </div>{" "}
          </>
        ) : (
          <span className="chatbox-empty-message">
            Tap on a chat to start conversation...
          </span>
        )}
      </div>
    </>
  );
};

export default ChatBox;
```

**Monolith (Monolito):** El componente `ChatBox` podría considerarse un "monolito" ya que encapsula la funcionalidad completa de un cuadro de chat, incluyendo la lógica de mensajes y la interfaz de usuario.

**Cookbook (Recetario):** El uso de componentes predefinidos como `InputEmoji`, la representación visual de mensajes y el manejo de eventos de cambio y clic reflejan un estilo de "recetario".

**Hollywood:** El manejo de la lógica de mensajes, la obtención de usuarios y la comunicación con el servidor a través de API muestran cierta inversión de control.

**Abstract Things (Cosas Abstractas):** El componente `ChatBox` encapsula la lógica para mostrar mensajes, manejar la entrada de texto y la interacción con emojis.

**Infinite Mirror (Espejo Infinito):** Podría haber múltiples instancias del componente `ChatBox` en la aplicación, lo que podría reflejar una replicación similar.

**Pipeline (Tubería):** La lógica de obtención de usuarios, obtención de mensajes, manejo de mensajes y renderizado de la interfaz se puede ver como pasos en un flujo de datos.

**Code Golf (Golf de código):** El código es conciso y utiliza variables para mantener la claridad del flujo de lógica, sin comprometer la legibilidad.

**The One (El Elegido):** El componente `ChatBox` podría considerarse el "Elegido" para coordinar y gestionar la representación visual de la conversación en el chat.

---

### Componente `FollowersCard`

```jsx
import React, { useState } from "react";
import "./FollowersCard.css";
import FollowersModal from "../FollowersModal/FollowersModal";
import { getAllUser } from "../../api/UserRequests";
import User from "../User/User";
import { useSelector } from "react-redux";
const FollowersCard = ({ location }) => {
  const [modalOpened, setModalOpened] = useState(false);
  const [persons, setPersons] = useState([]);
  const { user } = useSelector((state) => state.authReducer.authData);

  useEffect(() => {
    const fetchPersons = async () => {
      const { data } = await getAllUser();
      setPersons(data);
    };
    fetchPersons();
  }, []);

  return (
    <div className="FollowersCard">
      <h3>People you may know</h3>

      {persons.map((person, id) => {
        if (person._id !== user._id) return <User person={person} key={id} />;
      })}
      {!location ? (
        <span onClick={() => setModalOpened(true)}>Show more</span>
      ) : (
        ""
      )}

      <FollowersModal
        modalOpened={modalOpened}
        setModalOpened={setModalOpened}
      />
    </div>
  );
};

export default FollowersCard;
```

**Monolith (Monolito):** El componente `FollowersCard` no es necesariamente un monolito, ya que se centra en mostrar sugerencias de personas y enlaces de navegación.

**Cookbook (Recetario):** El uso de componentes predefinidos como `Modal` y `FollowersCard`, así como la reutilización de patrones para manejar la apertura del modal y mostrar más sugerencias, reflejan un estilo de "recetario".

**Hollywood:** El manejo de la lógica de abrir y cerrar el modal está invertido y se delega en componentes externos como `Modal` y `FollowersModal`.

**Abstract Things (Cosas Abstractas):** El componente `FollowersCard` encapsula la lógica para mostrar sugerencias de personas y utiliza componentes abstractos como `Modal` y enlaces.

**Pipeline (Tubería):** La lógica de apertura del modal y la obtención de sugerencias de personas se pueden ver como pasos en un flujo de datos.

**Code Golf (Golf de código):** El código es conciso y utiliza componentes de biblioteca para representar sugerencias de personas y manejar la apertura del modal.

**The One (El Elegido):** El componente `FollowersCard` podría considerarse el "Elegido" para coordinar y gestionar la representación de sugerencias de personas.

---

### Componente `FollowersModal`

```jsx
import React from "react";
import { Modal, useMantineTheme } from "@mantine/core";
import FollowersCard from "../FollowersCard/FollowersCard";

const FollowersModal = ({ modalOpened, setModalOpened }) => {
  const theme = useMantineTheme();
  return (
    <Modal
      overlayColor={
        theme.colorScheme === "dark"
          ? theme

.colors.dark[9]
          : theme.colors.gray[2]
      }
      overlayOpacity={0.55}
      overlayBlur={3}
      size="55%"
      opened={modalOpened}
      onClose={() => setModalOpened(false)}
    >

    <FollowersCard location='modal'/>
    </Modal>
  );
};

export default FollowersModal;
```

**Monolith (Monolito):** El componente `FollowersModal` no es un monolito en sí mismo, ya que está relacionado con la representación de modales y la interacción con la biblioteca `@mantine/core`.

**Cookbook (Recetario):** Utiliza la biblioteca `@mantine/core` para establecer propiedades del modal y contiene un componente `FollowersCard` predefinido.

**Hollywood:** Delega la lógica del modal y su apertura/cierre en la biblioteca `@mantine/core` y en el componente `FollowersCard`.

**Abstract Things (Cosas Abstractas):** El componente `FollowersModal` encapsula la lógica para mostrar un modal y utiliza la abstracción proporcionada por la biblioteca `@mantine/core`.

**Code Golf (Golf de código):** El código es conciso y utiliza propiedades predefinidas de la biblioteca `@mantine/core`.

**The One (El Elegido):** El componente `FollowersModal` podría considerarse el "Elegido" para representar la interacción con modales en la aplicación.

---

### Componente `Post`

```jsx
import React, { useState } from "react";
import "./Post.css";
import Comment from "../../img/comment.png";
import Share from "../../img/share.png";
import Heart from "../../img/like.png";
import NotLike from "../../img/notlike.png";
import { likePost } from "../../api/PostsRequests";
import { useSelector } from "react-redux";

const Post = ({ data }) => {
  const { user } = useSelector((state) => state.authReducer.authData);
  const [liked, setLiked] = useState(data.likes.includes(user._id));
  const [likes, setLikes] = useState(data.likes.length);

  const handleLike = () => {
    likePost(data._id, user._id);
    setLiked((prev) => !prev);
    liked ? setLikes((prev) => prev - 1) : setLikes((prev) => prev + 1);
  };

  return (
    <div className="Post">
      <img
        src={data.image ? process.env.REACT_APP_PUBLIC_FOLDER + data.image : ""}
        alt=""
      />

      <div className="postReact">
        <img
          src={liked ? Heart : NotLike}
          alt=""
          style={{ cursor: "pointer" }}
          onClick={handleLike}
        />
        <img src={Comment} alt="" />
        <img src={Share} alt="" />
      </div>

      <span style={{ color: "var(--gray)", fontSize: "12px" }}>
        {likes} likes
      </span>
      <div className="detail">
        <span>
          <b>{data.name} </b>
        </span>
        <span>{data.desc}</span>
      </div>
    </div>
  );
};

export default Post;
```

**Monolith (Monolito):** El componente `Post` no es un monolito en sí mismo, ya que representa un fragmento específico de la interfaz de usuario relacionado con la representación de publicaciones y la interacción de "me gusta".

**Cookbook (Recetario):** Utiliza imágenes predefinidas para representar elementos visuales como iconos y muestra la cantidad de "me gusta" y comentarios.


**Abstract Things (Cosas Abstractas):** El componente `Post` encapsula la lógica para manejar "me gusta" y la representación visual de la publicación.

**Code Golf (Golf de código):** El código es relativamente conciso y utiliza operaciones condicionales para manejar el estado de "me gusta".

**The One (El Elegido):** El componente `Post` podría considerarse el "Elegido" para representar la interacción de "me gusta" y la visualización de publicaciones en la aplicación.

---

### Componente `NavIcons`

```jsx
import React from "react";
import { UilSetting } from "@iconscout/react-unicons";
import { Link } from "react-router-dom";
import "./Navicons.css";
import { AiFillHome, AiFillBell } from 'react-icons/ai';
import { BsFillGearFill, BsFillChat

DotsFill } from 'react-icons/bs';

const NavIcons = () => {
    return (
        <div className="navIcons">
            <Link to="../home">
                <AiFillHome className="unsa"/>
            </Link>
            <BsFillGearFill className="unsa"/>
            <AiFillBell className="unsa"/>
            <Link to="../chat">
                <BsFillChatDotsFill className="unsa"/>
            </Link>
        </div>
    );
};

export default NavIcons;
```

**Monolith (Monolito):** El componente `NavIcons` no es un monolito en sí mismo, ya que representa un fragmento específico de la interfaz de usuario relacionado con los íconos de navegación.

**Cookbook (Recetario):** Utiliza íconos predefinidos de bibliotecas como `react-icons` y `@iconscout/react-unicons` para representar íconos de navegación.

**Abstract Things (Cosas Abstractas):** El componente `NavIcons` encapsula la lógica para mostrar íconos de navegación y utiliza componentes abstractos para representar elementos visuales.

**Code Golf (Golf de código):** El código es relativamente conciso y utiliza componentes de biblioteca para representar íconos de navegación.

**The One (El Elegido):** El componente `NavIcons` podría considerarse el "Elegido" para representar la parte de la interfaz de usuario relacionada con los íconos de navegación en la aplicación.

---
