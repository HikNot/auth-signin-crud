# auth-signin-crud

## Перед использованием ФОРКНУТЬ!

!! в терминале ubuntu пишем sudo service postgresql start <br>
!! указываем пароль от супер-пользователя <br>
!! в каждой папке прописывем перед началом работы прописываем <br>
!! если у вас другие нейминги файлов и стру

```
npm i
```

// Шаг 0

в корне проекта в терминал прописываем

```
npx degit grentank/create-prettierrc --force
```

// Шаг 1

в терминале пишем

```
create vite@latest
```

-> client
-> React
-> JS + SWC

// Шаг 2

устанавливаем все нужные пакеты

```
npm i react-bootstrap bootstrap react-router-dom axios react-loader-spinner react-icons
```

// Шаг 3 (Client)

заходим в папку client

```
cd client/
```

в клиенте создаем папку src -> components -> pages, ui, hoc и файл Layout.jsx

в hoc создаем файл ProtectedRoute.jsx и Loader.jsx

ProtectedRouter.jsx

```
import { Navigate, Outlet } from 'react-router-dom';
import React from 'react';

export default function ProtectedRoute({
  children,
  isAllowed,
  redirectPath = '/',
}) {
  if (!isAllowed) return <Navigate to={redirectPath} replace />;
  return children || <Outlet />;
}

```

Loader.jsx

```
import React, { useEffect } from 'react';
import { Col, Row } from 'react-bootstrap';
import Spinner from '../ui/Spinner';

export default function Loader({ children, isLoading }) {
  useEffect(() => {
    if (isLoading) {
      document.body.style.overflow = 'hidden';
    } else {
      document.body.style.overflow = '';
    }

    return () => {
      document.body.style.overflow = '';
    };
  }, [isLoading]);

  return isLoading ? (
    <Row className="vh-100 d-flex justify-content-center align-items-center">
      <Col xs={2}>
        <Spinner />
      </Col>
    </Row>
  ) : (
    children
  );
}

```

в src создаем файл axiosInstance.js и пишем

```
import axios from 'axios';

const axiosInstance = axios.create({
  baseURL: '/api',
  headers: {
    'Content-Type': 'application/json',
  },
});

let accessToken = '';

export function setAccessToken(newToken) {
  accessToken = newToken;
}

axiosInstance.interceptors.request.use((config) => {
  if (!config.headers.Authorization) {
    config.headers.Authorization = `Bearer ${accessToken}`;
  }
  return config;
});

axiosInstance.interceptors.response.use(
  (res) => res,
  async (err) => {
    const prevRequest = err.config;
    if (err.response.status === 403 && !prevRequest.sent) {
      prevRequest.sent = true;
      const { data } = await axios('/api/tokens/refresh');
      accessToken = data.accessToken;
      prevRequest.headers.Authorization = `Bearer ${accessToken}`;
      return axiosInstance(prevRequest);
    }
    return Promise.reject(err);
  },
);

export default axiosInstance;
```

Layout.jsx

```
import React from 'react';
import NavBar from './ui/NavBar';
import { Outlet } from 'react-router-dom';
import Container from 'react-bootstrap/Container';
import Loader from './hoc/Loader';

export default function Layout({ user, handleLogout }) {
  return (
    <>
      <Loader isLoading={user === undefined}>
        <NavBar user={user} handleLogout={handleLogout} />
        <Outlet />
      </Loader>
    </>
  );
}


```

// Шаг 4 (Client)

создаем MainPage.jsx и пишем

```
import React, { useEffect, useState } from 'react';
import axiosInstance from '../../axiosInstance';
import PostCard from '../ui/PostCard';
import Container from 'react-bootstrap/Container';
import Row from 'react-bootstrap/Row';
import Col from 'react-bootstrap/Col';

export default function MainPage({ user }) {
  const [posts, setPosts] = useState([]);

  const handleClick = async (id) => {
    const res = await axiosInstance.post(`/likes/${id}`);
    setPosts((prev) => prev.map((el) => (el.id === id ? res.data : el)));
  };

  useEffect(() => {
    axiosInstance.get('/posts').then((res) => setPosts(res.data));
  }, []);

  const moreInfoHandler = async (id) => {
    console.log(id);
  }

  return (
    <>
      <Container>
        <Row className="mt-1">
          {posts.map((post) => (
            <Col className="mt-3 mb-3" key={post.id}>
              <PostCard handleClick={handleClick} user={user} post={post} moreInfoHandler={moreInfoHandler}/>
            </Col>
          ))}
        </Row>
      </Container>
    </>
  );
}


```

в папке pages создаем файл SignUpPage.jsx

```
import React from 'react';
import Form from 'react-bootstrap/Form';
import Button from 'react-bootstrap/Button';
import Container from 'react-bootstrap/Container';

export default function SignupPage({ handleSignUp }) {
  return (
    <Container
      style={{
        display: 'flex',
        justifyContent: 'center',
        alignItems: 'center',
        height: '100svh',
      }}
    >
      <Form onSubmit={handleSignUp}>
        <Form.Group>
          <Form.Label><h3>Username</h3></Form.Label>
          <Form.Control
            name="name"
            className="mb-3"
            type="text"
            placeholder="Enter username"
          />
        </Form.Group>

        <Form.Group className="block mar-b-1" controlId="formBasicEmail">
          <Form.Label><h3>Email address</h3></Form.Label>
          <Form.Control
            name="email"
            className="mb-3"
            type="email"
            placeholder="Enter email"
          />
        </Form.Group>

        <Form.Group className="block mar-b-1" controlId="formBasicPassword">
          <Form.Label><h3>Password</h3></Form.Label>
          <Form.Control
            name="password"
            className="mb-3"
            type="password"
            placeholder="Password"
          />
        </Form.Group>

        <Form.Group className="block mar-b-1" controlId="formConfirmPassword">
          <Form.Label><h3>Confirm password</h3></Form.Label>
          <Form.Control
            className="mb-3"
            type="password"
            placeholder="Password"
          />
        </Form.Group>
        <Button variant="primary" type="submit">
          Submit
        </Button>
      </Form>
    </Container>
  );
}
```

создаем файл SignInPage.jsx

```
import React from 'react';
import Form from 'react-bootstrap/Form';
import Button from 'react-bootstrap/Button';
import Container from 'react-bootstrap/Container';

export default function SignInPage({handleSignIn}) {
  return (
    <Container style={{
      display: 'flex',
      justifyContent: 'center',
      alignItems: 'center',
      height: '100svh',
    }}>
      <Form onSubmit={handleSignIn}>
        <Form.Group className="block mar-b-1" controlId="formBasicEmail">
          <Form.Label><h1>E-mail</h1></Form.Label>
          <Form.Control
            name="email"
            className="mb-3"
            type="email"
            placeholder="Enter email"
          />
        </Form.Group>

        <Form.Group className="block mar-b-1" controlId="formBasicPassword">
          <Form.Label><h1>Пароль</h1></Form.Label>
          <Form.Control
            name="password"
            className="mb-3"
            type="password"
            placeholder="Password"
          />
        </Form.Group>

        <Button variant="primary" type="submit">
          Войти
        </Button>
      </Form>
    </Container>
  );
}

```

AccountPage.jsx

```
import React, { useEffect, useState } from 'react';
import Container from 'react-bootstrap/Container';
import Row from 'react-bootstrap/Row';
import Col from 'react-bootstrap/Col';
import Figure from 'react-bootstrap/Figure';
import FloatingLabel from 'react-bootstrap/FloatingLabel';
import Form from 'react-bootstrap/Form';
import axiosInstance from '../../axiosInstance';
import Button from 'react-bootstrap/Button';
import PostCard from '../ui/PostCard';
import EditModal from '../ui/Modal';

export default function AccountPage({ user }) {
  const [post, setPost] = useState([]);
  const [addPost, setAddPost] = useState([]);
  const [modalContent, setModalContent] = useState(null);

  useEffect(() => {
    axiosInstance.get(`/posts/${user.id}`).then((res) => setPost(res.data));
  }, []);

  const addPostHandler = async (formData) => {
    const res = await axiosInstance.post(`/posts/${user.id}`, formData);
    if (res.status === 201) {
      alert('Пост успешно добавлен');
      setAddPost((prev) => [...prev, res.data]);
      window.location.reload();
    }
  };

  const deletePostHandler = async (id) => {
    const res = await axiosInstance.delete(`posts/${id}`);
    if (res.status === 200) {
      alert('Пост успешно удален');
      setAddPost((prev) => prev.filter((el) => el.id !== id));
      window.location.reload();
    }
  };

  const editHandler = async (e) => {
    e.preventDefault();
    const formData = Object.fromEntries(new FormData(e.target));
    const res = await axiosInstance.patch(
      `/posts/${modalContent.id}`,
      formData,
    );
    setPost((prev) =>
      prev.map((el) => (el.id === modalContent.id ? res.data : el)),
    );
    setModalContent(null);
  };

  return (
    <Container>
      <Row xs={2} md={4} lg={6}>
        <Col>
          <Figure>
            <Figure.Image
              width={171}
              height={180}
              alt="171x180"
              src="https://cdn.icon-icons.com/icons2/1812/PNG/512/4213460-account-avatar-head-person-profile-user_115386.png"
            />
            <Figure.Caption>Пока в разработке</Figure.Caption>
          </Figure>
        </Col>
        <Col className='mt-4'>
          <h1>{user.name}</h1>
          <h3>{user.email}</h3>
        </Col>
      </Row>
      <Row xs={1} md={2} className="mt-1">
        {post.map((el) => (
          <Col key={el.id} className="mt-3">
            <PostCard
              setModalContent={setModalContent}
              deletePostHandler={deletePostHandler}
              user={user}
              post={el}
            />
          </Col>
        ))}
      </Row>
      <Row xs="auto">
        <Col className="mt-3">
          <h2>Добавить новый пост</h2>{' '}
          <Form
            onSubmit={(e) => {
              e.preventDefault();
              const formData = Object.fromEntries(new FormData(e.target));
              addPostHandler(formData);
            }}
          >
            <FloatingLabel
              controlId="floatingInput"
              label="Заголовок поста"
              className="mb-3"
            >
              <Form.Control
                name="title"
                type="text"
                placeholder="Введите заголовок поста"
              />
            </FloatingLabel>
            <FloatingLabel controlId="floatingPassword" label="Текст поста">
              <Form.Control
                name="body"
                type="text"
                placeholder="Введите текст поста"
              />
            </FloatingLabel>
            <Button type="submit" className="mt-3 mb-3" variant="outline-info">
              Добавить
            </Button>{' '}
          </Form>
        </Col>
      </Row>
      <EditModal
        modalContent={modalContent}
        setModalContent={setModalContent}
        editHandler={editHandler}
      />
    </Container>
  );
}

```

WelcomePage.jsx

```
import React from 'react';
import { NavLink } from 'react-router-dom';

export default function WelcomePage() {
  return (
    <>
      <div
        style={{
            color: 'black',
          display: 'flex',
          flexDirection: 'column',
          justifyContent: 'center',
          alignItems: 'center',
          marginTop: '250px',
        }}
      >
        <h1>Здравствуй пользователь!</h1>
        <br/>
        <br/>
        <NavLink
          to={'/signup'}
          style={{
            textDecoration: 'none',
            color: 'black',
            fontSize: '20px',
            fontWeight: 'bold',
          }}
        >
          <h3>Зарегистрироваться</h3>
        </NavLink>
        <NavLink
          to={'/signin'}
          style={{
            textDecoration: 'none',
            color: 'black',
            fontSize: '20px',
            fontWeight: 'bold',
          }}
        >
          <h3>Войти</h3>
        </NavLink>
      </div>
    </>
  );
}

```

PostPage.jsx

```
import React, { useEffect, useState } from 'react';
import axiosInstance from '../../axiosInstance';
import { useParams } from 'react-router-dom';
import Container from 'react-bootstrap/Container';
import Row from 'react-bootstrap/Row';
import Col from 'react-bootstrap/Col';
import PostCard from '../ui/PostCard';
import FloatingLabel from 'react-bootstrap/FloatingLabel';
import Form from 'react-bootstrap/Form';
import { Button } from 'react-bootstrap';
import CommentsCard from '../ui/CommentsCard';

export default function PostPage() {
  const [posts, setPosts] = useState(null);
  const [comments, setComments] = useState([]);
  const { id } = useParams();

  const addComment = async (formdata) => {
    const res = await axiosInstance.post(`/comments/${id}`, [
      formdata,
      posts.id,
    ]);
    console.log(res.data);
  };
  useEffect(() => {
    axiosInstance
      .get(`/posts/one-post/${id}`)
      .then((res) => setPosts(res.data));
  }, []);
  useEffect(() => {
    axiosInstance.get(`/comments/${id}`).then((res) => setComments(res.data));
  }, []);

  if (!posts) return <h2>Loading....</h2>;

  return (
    <Container>
      <Row xs={4} md={6} lg={8} className="mt-1">
        <Col key={posts.id}>
          <PostCard post={posts} />
        </Col>
      </Row>
      <Row xs={1} md={2} className="mt-3">
        <Col>
          <h3>Оставить комментарий</h3>
          <Form
            onSubmit={(e) => {
              e.preventDefault();
              const formData = Object.fromEntries(new FormData(e.target));
              addComment(formData);
            }}
          >
            <FloatingLabel
              controlId="floatingInput"
              label="Ваш комментарий здесь"
              className="mb-3"
            >
              <Form.Control
                name="body"
                type="text"
                placeholder="Ваш комментарий здесь"
              />
            </FloatingLabel>
            <Button type="submit" className="mb-3" variant="outline-info">
              Добавить
            </Button>{' '}
          </Form>
        </Col>
      </Row>
      <Row style={{width: "60%"}}>
        {comments.map((comment) => (
          <Col key={comment.id}>
            <CommentsCard comment={comment} />
          </Col>
        ))}
      </Row>
    </Container>
  );
}

```

ChatPage.jsx

```
import React, { useEffect, useRef, useState } from 'react';
import Container from 'react-bootstrap/Container';
import ChatList from '../ui/ChatList';
import Row from 'react-bootstrap/Row';
import Col from 'react-bootstrap/Col';
import ChatComponent from '../ui/ChatComponent';

export default function ChatPage({user}) {
  const [online, setOnline] = useState([]);
  const [messages, setMessages] = useState([]);
  const soketRef = useRef(null);

  useEffect(() => {
    const soket = new WebSocket('http://localhost:3000');
    soketRef.current = soket;
    soket.onopen = () => console.log('CONNECTED');
    soket.onclose = () => console.log('DISCONNECTED');
    soket.onerror = () => console.log('ERROR');
    soket.onmessage = (event) => {
      const action = JSON.parse(event.data);
      const { type, payload } = action;
      switch (type) {
        case 'SET_USERS':
          setOnline(payload);
          break;
        case 'NEW_MESSAGES':
          setMessages(payload);
          break;
        case 'ADD_MESSAGE':
          setMessages((prev) => [...prev, payload]);
          break;
        default:
          break;
      }
    };
    return () => {
      soketRef.current.close();
    };
  }, []);

  const sendNewMessage = (text) => {
    const action = {
      type: 'NEW_MESSAGE',
      payload: {
        text,
      },
    };
    soketRef.current.send(JSON.stringify(action));
  };

  return (
    <Container>
      <Row>
        <Col className="mt-3" xs={2}>
          <ChatList online={online} />
        </Col>
        <Col className="mt-3" xs={10}>
          <ChatComponent user={user} sendNewMessage={sendNewMessage} messages={messages} />
        </Col>
      </Row>
    </Container>
  );
}

```

в ui создаем файлы Modal.jsx, NavBar.jsx, PostCard.jsx, Spiner.jsx, CommentsCard.jsx

Modal.jsx

```
import React from 'react';
import Button from 'react-bootstrap/Button';
import Modal from 'react-bootstrap/Modal';
import FloatingLabel from 'react-bootstrap/FloatingLabel';
import Form from 'react-bootstrap/Form';

export default function EditModal({
  setModalContent,
  modalContent,
  editHandler,
}) {
  const handleClose = () => {
    setModalContent(null);
  };
  return (
    <Modal show={!!modalContent} onHide={handleClose}>
      <Modal.Header closeButton>
        <Modal.Title>Редактирование поста</Modal.Title>
      </Modal.Header>
      <Modal.Body>
        <Form onSubmit={editHandler}>
          <FloatingLabel
            controlId="floatingInput"
            label="Заголовок поста"
            className="mb-3"
          >
            <Form.Control
              name="title"
              type="text"
              placeholder="Введите заголовок поста"
              defaultValue={modalContent?.title}
            />
          </FloatingLabel>
          <FloatingLabel controlId="floatingPassword" label="Текст поста">
            <Form.Control
              name="body"
              type="text"
              placeholder="Введите текст поста"
              defaultValue={modalContent?.body}
            />
          </FloatingLabel>
      <Button className='mt-3' variant="primary" type='submit'>
        Save Changes
      </Button>
        </Form>
      </Modal.Body>
    </Modal>
  );
}

```

NavBar.jsx

```
import React from 'react';
import Navbar from 'react-bootstrap/Navbar';
import Container from 'react-bootstrap/Container';
import { NavLink } from 'react-router-dom';

export default function NavBar({ user, handleLogout }) {
  return (
    <Navbar className="bg-body-tertiary">
      <Container>
        <NavLink style={{ textDecoration: 'none' }} to={'/'}>
          <Navbar.Brand>Solo library</Navbar.Brand>
        </NavLink>
        <Navbar.Toggle />
        <Navbar.Collapse className="justify-content-end">
          {user ? (
            <div>
              <Navbar.Text className="me-3">
                <NavLink to="/home">Home</NavLink>
              </Navbar.Text>
              <Navbar.Text className="me-3">
                <NavLink to="/my-cards">My cards</NavLink>
              </Navbar.Text>
              <Navbar.Text className="me-3">
                <NavLink to="/chat">Chat</NavLink>
              </Navbar.Text>
              <Navbar.Text className="me-3">
                <NavLink to="/" onClick={handleLogout}>
                  Logout
                </NavLink>
              </Navbar.Text>
            </div>
          ) : (
            <div>
              <Navbar.Text className="me-3">
                <NavLink to="/signup">SignUp</NavLink>
              </Navbar.Text>
              <Navbar.Text className="me-3">
                <NavLink to="/signin">SignIn</NavLink>
              </Navbar.Text>
            </div>
          )}
          <Navbar.Text>
            {user ? `Signed in as: ${user.name}!` : 'Signed in as: Guest'}
          </Navbar.Text>
        </Navbar.Collapse>
      </Container>
    </Navbar>
  );
}


```

PostCard.jsx

```
import React from 'react';
import Card from 'react-bootstrap/Card';
import ListGroup from 'react-bootstrap/ListGroup';
import Button from 'react-bootstrap/Button';
import { FcLike } from 'react-icons/fc';
import { useNavigate } from 'react-router-dom';

export default function PostCard({
  post,
  deletePostHandler,
  setModalContent,
  user,
  handleClick,
  moreInfoHandler,
}) {

  console.log(post);
  const truncateText = (text, maxLength) => {
    if (text.length > maxLength) {
      return text.slice(0, maxLength) + '...';
    }
    return text;
  };

  const navigate = useNavigate();

  return (
    <Card style={{ width: '18rem' }}>
      <Card.Img
        variant="top"
        src="https://w7.pngwing.com/pngs/124/1016/png-transparent-new-post-thumbnail.png"
      />
      <Card.Body>
        <Card.Title>{post?.title}</Card.Title>
        <Card.Text>{truncateText(post?.body, 30)}</Card.Text>
      </Card.Body>
      <ListGroup className="list-group-flush">
        <ListGroup.Item>Автор: {post.User?.name}</ListGroup.Item>
      </ListGroup>
      <Card.Body>
        <Button onClick={() => deletePostHandler(post.id)} variant="danger">
          Удалить
        </Button>{' '}
        <Button onClick={() => setModalContent(post)} variant="warning">
          Редактировать
        </Button>{' '}
        <Button
          onClick={() => navigate(`/${post.id}`)}
          variant="info"
          className="mt-1"
        >
          Подробнее
        </Button>
        <span
          onClick={() => handleClick(post.id)}
          data-testid={`likes-${post.likes}`}
          style={{
            cursor: 'pointer',
          }}
        >
          <FcLike />
          <span data-testid={post.id} className="ms-1">
            {post.Likes?.length}
          </span>
        </span>
      </Card.Body>
    </Card>
  );
}
```

Spinner.jsx

```
import React from 'react';
import { TailSpin } from 'react-loader-spinner';

export default function Spinner() {
  return (
    <TailSpin
      visible={true}
      height="80"
      width="80"
      color="#A8A8A8"
      ariaLabel="tail-spin-loading"
      radius="1"
      wrapperStyle={{}}
      wrapperClass=""
    />,
  );
}

```

CommentsCard.jsx

```
import React from 'react';
import Card from 'react-bootstrap/Card';

export default function CommentsCard(comment) {
    console.log(comment.comment.User.name);
  return (
    <Card>
      <Card.Body><h2>{comment.comment?.body}</h2></Card.Body>
      <Card.Body><h3>{comment.comment?.User.name}</h3></Card.Body>
    </Card>
  );
}

```

ChatComponent.jsx

```
import React from 'react';
import Row from 'react-bootstrap/Row';
import Col from 'react-bootstrap/Col';
import ChatMessage from './ChatMessage';
import Form from 'react-bootstrap/Form';
import { IoSend } from 'react-icons/io5';
import ChatInput from './ChatInput';

export default function ChatComponent({ user, messages, sendNewMessage }) {
  return (
    <>
      <Row>
        <Col xs={12}>
          {messages.map((message) => (
            <ChatMessage user={user} message={message} key={message.id} />
          ))}
        </Col>
        <Col>
          <ChatInput sendNewMessage={sendNewMessage} />
        </Col>
      </Row>
    </>
  );
}

```

ChatInput.jsx

```
import React from 'react';
import Row from 'react-bootstrap/Row';
import Col from 'react-bootstrap/Col';
import Form from 'react-bootstrap/Form';
import { IoSend } from 'react-icons/io5';

export default function ChatInput({ sendNewMessage }) {
  return (
    <Form onSubmit={(e) => {
        e.preventDefault();
        const form = e.target
        const text = new FormData(form).get("text")
        sendNewMessage(text)
        form.reset()
    }}>
      <Row>
        <Col className="mt-3" xs={9}>
          <Form.Control name="text" type="text" placeholder="Enter your message" />
        </Col>
        <Col xs={3} style={{ marginTop: '1.25rem' }}>
          <IoSend
            style={{ cursor: 'pointer' }}
            onClick={() => console.log('clicked')}
          />
        </Col>
      </Row>
    </Form>
  );
}

```

ChatList.jsx

```
import React from 'react';
import ListGroup from 'react-bootstrap/ListGroup';

export default function ChatList({ online }) {
  console.log(online);

  return (
    <ListGroup>
      {online.map((user) => (
        <ListGroup.Item key={user.id}>{user.name}</ListGroup.Item>
      ))}
    </ListGroup>
  );
}

```

ChatMessage.jsx

```
import React from 'react';
import Col from 'react-bootstrap/esm/Col';
import Row from 'react-bootstrap/esm/Row';

export default function ChatMessage({ user, message }) {
  // console.log(messages[0].User.name);
  console.log(message);

  return (
    <div
      style={{
        display: 'flex',
        justifyContent: user.id === message.userId ? 'flex-end' : 'flex-start',
      }}
    >
      {message.body}&nbsp;&nbsp;<i>-{message.User?.name}</i>
    </div>
  );
}

```

В App.jsx прописыаем

```
import React, { useEffect, useState } from 'react';
import Layout from './components/Layout';
import MainPage from './components/pages/MainPage';
import {
  createBrowserRouter,
  Navigate,
  RouterProvider,
  useNavigate,
} from 'react-router-dom';
import SignUpPage from './components/pages/SignUpPage';
import axiosInstance, { setAccessToken } from './axiosInstance';
import SignInPage from './components/pages/SignInPage';
import ProtectedRoute from './components/hoc/ProtectedRoute';
import WelcomePage from './components/pages/WelcomePage';
import AccountPage from './components/pages/AccountPage';
import PostPage from './components/pages/PostPage';
import ChatPage from './components/pages/ChatPage';

function App() {
  const [user, setUser] = useState();

  useEffect(() => {
    const time = setTimeout(
      () =>
        axiosInstance
          .get('/tokens/refresh')
          .then((res) => {
            const { user, accessToken } = res.data;
            setUser(user);
            setAccessToken(accessToken);
          })
          .catch(() => {
            setUser(null);
            setAccessToken('');
          }),
      1000,
    );

    return () => clearTimeout(time);
  }, []);

  const handleSignUp = async (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const data = Object.fromEntries(formData);
    const res = await axiosInstance.post('/auth/signup', data);
    if (res.status === 200) {
      setUser(res.data.user);
      setAccessToken(res.data.accessToken);
    }
  };

  const handleSignIn = async (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const data = Object.fromEntries(formData);
    const res = await axiosInstance.post('/auth/signin', data);
    if (res.status === 200) {
      setUser(res.data.user);
      setAccessToken(res.data.accessToken);
    }
    window.location.href = '/home';
  };

  const handleLogout = async () => {
    const res = await axiosInstance.post('/auth/logout');
    if (res.status === 200) {
      setUser(null);
      setAccessToken('');
    }
  };

  const router = createBrowserRouter([
    {
      element: <Layout user={user} handleLogout={handleLogout} />,
      children: [
        {
          path: '/',
          element: <Navigate to="/home" replace />,
        },
        {
          path: '/home',
          element: (
            <ProtectedRoute isAllowed={!!user} redirectPath="/welcome">
              <MainPage user={user}></MainPage>
            </ProtectedRoute>
          ),
        },
        {
          path: '/my-cards',
          element: (
            <ProtectedRoute isAllowed={!!user} redirectPath="/welcome">
              <AccountPage user={user} />
            </ProtectedRoute>
          ),
        },
        {
          path: '/chat',
          element: (
            <ProtectedRoute isAllowed={!!user} redirectPath="/welcome">
              <ChatPage user={user}/>
            </ProtectedRoute>
          ),
        },
        {
          path: '/:id',
          element: (
            <ProtectedRoute isAllowed={!!user} redirectPath="/welcome">
              <PostPage />
            </ProtectedRoute>
          ),
        },
        {
          element: <ProtectedRoute isAllowed={!user} redirectPath="/" />,
          children: [
            {
              path: '/welcome',
              element: <WelcomePage />,
            },
            {
              path: '/signup',
              element: <SignUpPage handleSignUp={handleSignUp} />,
            },
            {
              path: '/signin',
              element: <SignInPage handleSignIn={handleSignIn} />,
            },
          ],
        },
      ],
    },
  ]);

  return <RouterProvider router={router} />;
}

export default App;

```

в main.jsx

```
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import 'bootstrap/dist/css/bootstrap.min.css';

ReactDOM.createRoot(document.getElementById('root')).render(
 // <React.StrictMode>
    <App />
 //  </React.StrictMode>,
)

```

в vite.config.js

```
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react-swc'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': 'http://localhost:3000'
    }
  }
})
```

// Шаг 5 (Server)

выходим в корень проекта

```
cd ..
```

и заходим в папку server

```
cd server/
```

доустанавливим зависимости

```
npm i sequelize pg pg-hstore express bcrypt cookie-parser dotenv jsonwebtoken morgan
```

```
npm i -D nodemon sequelize-cli
```

в файле package.json прописываем скрипты

```
"db": "sequelize-cli db:drop && sequelize-cli db:create && sequelize-cli db:migrate && sequelize-cli db:seed:all",
"dev": "nodemon src/server.js"
```

создаем файл .sequelizerc

```
const path = require('path');

module.exports = {
  config: path.resolve('config', 'database.js'),
  'models-path': path.resolve('db', 'models'),
  'seeders-path': path.resolve('db', 'seeders'),
  'migrations-path': path.resolve('db', 'migrations'),
};
```

создаем файлы .env и .env.example

.env.example

```
PORT=
DB_USER=
DB_NAME=
DB_HOST=
DB_PASS=
ACCESS_TOKEN_SECRET=
REFRESH_TOKEN_SECRET=
```

.env

```
PORT= // 3000
DB_USER= // ваш пользователь бд
DB_NAME= // любое имя для бд
DB_HOST= // 127.0.0.1
DB_PASS= // ваш пароль к бд
ACCESS_TOKEN_SECRET= // любой текст, цифры и спец. символы (без запятых точек и воскл. знаков)
REFRESH_TOKEN_SECRET= // любой текст, цифры и спец. символы (без запятых точек и воскл. знаков)
```

в папке config -> database.js

```
require('dotenv').config();

module.exports = {
  development: {
    username: process.env.DB_USER,
    password: process.env.DB_PASS,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    dialect: 'postgres',
  },
  test: {
    username: process.env.DB_USER,
    password: process.env.DB_PASS,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    dialect: 'postgres',
  },
  production: {
    username: 'root',
    password: null,
    database: 'database_production',
    host: '127.0.0.1',
    dialect: 'mysql',
  },
};
```

в терминале пишем

```
npx sequelize-cli init
```

```
npx sequelize-cli model:create --name User --attributes name:string,email:string,hashpass:string
```

по аналогии создаем столько моделей, сколько требует задание

создаем файл для генирации сидов

```
npx sequelize-cli seed:generate --name initial-seed
```

в файле сидов пишем

```
 await queryInterface.bulkInsert('Posts', [
    {
      title: 'Самый первый пост',
      body: 'Это самый первый пост',
      userId: null,
      createdAt: new Date(),
      updatedAt: new Date()
    },
    {
      title: 'Самый второй пост',
      body: 'Это самый второй пост',
      userId: null,
      createdAt: new Date(),
      updatedAt: new Date()
    }
  ], {});
  },
```

в папке models у юзера пишем

```
 class User extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate({ Post, Like, Comment }) {
      // define association here
      this.hasMany(X, { foreignKey: 'userId'});
      this.hasMany(Like, { foreignKey: 'userId'});
      this.hasMany(Comment, { foreignKey: 'userId'});
    }
  }
```

у модели X пишем

```
class X extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate({ User, Like, Comment }) {
      // define association here
      this.belongsTo(User, { foreignKey: 'userId' });
      this.hasMany(Like, { foreignKey: 'postId' });
      this.hasMany(Comment, { foreignKey: 'postId' })
    }
  }
```

// Для реализации лайков

у модели лайк создаем поля userId, postId <br><br>

в самой модели пишем

```
class Like extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate({ User, X }) {
      // define association here
      this.belongsTo(User, { foreignKey: 'userId' });
      this.belongsTo(X, { foreignKey: 'postId' });
    }
  }
```

у модели комментарии создаем поля body, userId, postId <br><br>

в самой модели пишем

```
class Comment extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate({User, Post}) {
      this.belongsTo(User, {foreignKey: 'userId'})
      this.belongsTo(Post, {foreignKey: 'postId'})
    }
  }
```

в миграциях у юзера пишем

```
'use strict';
/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('Users', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      name: {
        type: Sequelize.STRING
      },
      email: {
        type: Sequelize.STRING,
        unique: true
      },
      hashpass: {
        type: Sequelize.STRING
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable('Users');
  }
};
```

в миграциях модели X пишем

```
'use strict';
/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('Posts', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      title: {
        type: Sequelize.STRING
      },
      body: {
        type: Sequelize.STRING
      },
      userId: {
        type: Sequelize.INTEGER,
        references: {
          model: 'Users',
          key: 'id'
        }
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable('Posts');
  }
};
```

в миграциях у like пишем

```
'use strict';
/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('Likes', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      userId: {
        type: Sequelize.INTEGER,
        references: {
          model: 'Users',
          key: 'id'
        }
      },
      postId: {
        type: Sequelize.INTEGER,
        references: {
          model: 'Posts',
          key: 'id'
        },
        onDelete: 'CASCADE'
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable('Likes');
  }
};
```

в миграции comment пишем

```
'use strict';
/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('Comments', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      body: {
        type: Sequelize.TEXT,
        allowNull: false
      },
      userId: {
        type: Sequelize.INTEGER,
        allowNull: false,
        references: {
          model: 'Users',
          key: 'id'
        }
      },
      postId: {
        type: Sequelize.INTEGER,
        allowNull: false,
        references: {
          model: 'Posts',
          key: 'id'
        }
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable('Comments');
  }
};
```

### Для реалзицаии чата на web-soket`ах

создаем таблицу Message с полями body, userId <br><br>

в миграции пишем

```
'use strict';
/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('Messages', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      body: {
        type: Sequelize.TEXT
      },
      userId: {
        type: Sequelize.INTEGER,
        references: {
          model: 'Users',
          key: 'id'
        },
        onDelete: 'CASCADE'
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable('Messages');
  }
};
```

в модели пишем

```
'use strict';
const {
  Model
} = require('sequelize');
module.exports = (sequelize, DataTypes) => {
  class Message extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate({User}) {
      // define association here
      this.belongsTo(User, {foreignKey: 'userId'})
    }
  }
  Message.init({
    body: DataTypes.TEXT,
    userId: DataTypes.INTEGER
  }, {
    sequelize,
    modelName: 'Message',
  });
  return Message;
};
```


создаем папку src и в ней configs, middlewares, router, utils, ws, app.js и server.js

в папке configs создаем cookieConfig.js и jwtConfig.js

cookieConfig.js

```
const jwtConfig = require('./jwtConfig');

const cookieConfig = {
  httpOnly: true,
  maxAge: jwtConfig.refresh.expiresIn,
};

module.exports = cookieConfig;

```

jwtConfig.js

```
const jwtConfig = {
  access: {
    expiresIn: `${5 * 1000}`,
  },
  refresh: {
    expiresIn: `${12 * 60 * 60 * 1000}`,
  },
};

module.exports = jwtConfig;
```

в middlewares создаем файл verifyTokens.js

```
const jwt = require('jsonwebtoken');
require('dotenv').config();

function verifyRefreshToken(req, res, next) {
  try {
    const { refreshToken } = req.cookies;
    const { user } = jwt.verify(refreshToken, process.env.REFRESH_TOKEN_SECRET);
    res.locals.user = user;
    return next();
  } catch (error) {
    console.log(error);
    console.log('Invalid refresh token');
    return res.sendStatus(401);
  }
}

const verifyAccessToken = (req, res, next) => {
  try {
    const accessToken = req.headers.authorization.split(' ')[1]; // Bearer <token>
    const { user } = jwt.verify(accessToken, process.env.ACCESS_TOKEN_SECRET);
    res.locals.user = user;

    return next();
  } catch (error) {
    console.log('Invalid access token');
    return res.sendStatus(403);
  }
};

module.exports = {verifyRefreshToken, verifyAccessToken};
```

в папке router создаем файлы authRouter.js, postRouter.js (назвиние может меняться в соответствии с заданием), tokensRouter.js, userRouter.js, likeRouter.js и commentRouter.js

authRouter.js

```
const express = require('express');
const bcrypt = require('bcrypt');
const { User } = require('../../db/models');
const generateTokens = require('../utils/generateTokens');
const cookieConfig = require('../configs/cookieConfig');

const authRouter = express.Router();

authRouter.post('/signup', async (req, res) => {
  const { name, email, password } = req.body;
  if (!email && !name && !password) return res.sendStatus(400);

  const hashpass = await bcrypt.hash(password, 10);
  const [newUser, created] = await User.findOrCreate({
    where: { email },
    defaults: { name, hashpass },
  });
  if (!created) return res.sendStatus(400);

  const user = newUser.get();
  delete user.hashpass;
  const { accessToken, refreshToken } = generateTokens({ user });
  res
    .cookie('refreshToken', refreshToken, cookieConfig)
    .json({ accessToken, user });
});

authRouter.post('/signin', async (req, res) => {
  const { email, password } = req.body;
  if (!email || !password) return res.sendStatus(400);
  const foundUser = await User.findOne({ where: { email } });
  if (!foundUser) return res.sendStatus(400);
  const isValid = await bcrypt.compare(password, foundUser.hashpass);
  if (!isValid) return res.sendStatus(400);

  const user = foundUser.get();
  delete user.hashpass;
  const { accessToken, refreshToken } = generateTokens({ user });
  res
    .cookie('refreshToken', refreshToken, cookieConfig)
    .json({ accessToken, user });
});

authRouter.post('/logout', async (req, res) => {
  res.clearCookie('refreshToken').sendStatus(200);
});

module.exports = authRouter;

```

postRouter.js

```
const express = require('express');
const { User, Post } = require('../../db/models');
const postRouter = express.Router();
const { verifyAccessToken } = require('../middlewares/verifyRefreshToken');

postRouter.route('/').get(async (req, res) => {
  try {
    const posts = await Post.findAll({
      include: User,
    });
    res.status(200).json(posts);
  } catch (error) {
    console.log(error);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

postRouter
  .route('/:id')
  .get(async (req, res) => {
    const { id } = req.params;
    try {
      const post = await Post.findAll({ where: { userId: id } });
      res.status(200).json(post);
    } catch (err) {
      console.log(err);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  })
  .post(verifyAccessToken, async (req, res) => {
    const { title, body } = req.body;
    if (!title || !body) {
      return res.status(400).json({ error: 'Missing required fields' });
    }
    try {
      const newPost = await Post.create({
        title,
        body,
        userId: res.locals.user.id,
      });
      res.status(201).json(newPost);

      const plainPost = await Post.findOne({
        where: { id: newPost.id },
        include: {
          model: User,
          attributes: ['id', 'name', 'email'],
        },
      });
      res.json(plainPost);
    } catch (err) {
      console.log(err);
    }
  })
  .delete(verifyAccessToken, async (req, res) => {
    const { id } = req.params;
    try {
      const post = await Post.destroy({ where: { id } });
      res.status(200).json(post);
    } catch (err) {
      console.log(err);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  })
  .patch(verifyAccessToken, async (req, res) => {
    try {
      const { id } = req.params;

      const updateData = {};
      for (const key in req.body) {
        if (req.body[key]) {
          updateData[key] = req.body[key];
        }
      }

      await Post.update(updateData, { where: { id } });
      const updatedPost = await Post.findByPk(id);
      res.json(updatedPost);
    } catch (error) {
      res.status(500).json({ message: 'Server error' });
    }
  });

module.exports = postRouter;

```

tokensRouter.js

```
const express = require('express');
const { verifyRefreshToken } = require('../middlewares/verifyRefreshToken');
const generateTokens = require('../utils/generateTokens');
const cookieConfig = require('../configs/cookieConfig');

const tokenRouter = express.Router();

tokenRouter.get('/refresh', verifyRefreshToken, async (req, res) => {
  const { user } = res.locals;
  const { accessToken, refreshToken } = generateTokens({
    user,
  });

  res
    .cookie('refreshToken', refreshToken, cookieConfig)
    .json({ user, accessToken });
});

module.exports = tokenRouter;

```

userRouter.js

```
const express = require('express');
const { User } = require('../../db/models');
const userRouter = express.Router();

userRouter.route('/').get(async (req, res) => {
  try {
    const users = await User.findAll();
    res.status(200).json(users);
  } catch (error) {
    console.log(error);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

module.exports = userRouter;
```

likeRouter.js

```
const express = require('express');
const { Like, User, Post } = require('../../db/models');
const likeRouter = express.Router();
const { verifyAccessToken } = require('../middlewares/verifyRefreshToken');

likeRouter.route('/:id').post(verifyAccessToken, async (req, res) => {
  try {
    const existingLike = await Like.findOne({
      where: {
        userId: res.locals.user.id,
        postId: req.params.id,
      },
    });

    if (existingLike) {
      await Like.destroy({
        where: {
          userId: res.locals.user.id,
          postId: req.params.id,
        },
      });
      const post = await Post.findOne({
        where: { id: req.params.id },
        include: [User, Like],
      });
      return res.json(post);
    } else {
      const newLike = await Like.create({
        userId: res.locals.user.id,
        postId: req.params.id,
      });
      const post = await Post.findOne({
        where: { id: req.params.id },
        include: [User, Like],
      });
      return res.json(post);
    }
  } catch (error) {
    console.log(error);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

module.exports = likeRouter;

```

commentRouter.js

```
const express = require('express');
const { Like, User, Post, Comment } = require('../../db/models');
const commentRouter = express.Router();
const { verifyAccessToken } = require('../middlewares/verifyRefreshToken');

commentRouter
  .route('/:id')
  .get(async (req, res) => {
    const { id } = req.params;
    try {
      const comments = await Comment.findAll({
        where: { postId: id },
        include: User,
      });
      res.status(200).json(comments);
    } catch (error) {
      console.log(error);
      res.status(500).json({ message: 'Internal server error' });
    }
  })
  .post(verifyAccessToken, async (req, res) => {
    try {
      const comment = await Comment.create({
        body: req.body[0].body,
        userId: res.locals.user.id,
        postId: req.body[1],
      });
      res.status(201).json(comment);
    } catch (error) {
      console.log(error);
      res.status(500).json({ message: 'Internal server error' });
    }
  });

module.exports = commentRouter;

```

в папке utils создаем файл generateTokens.js

```
const jwt = require('jsonwebtoken');
const jwtConfig = require('../configs/jwtConfig');
require('dotenv').config();

function generateTokens(payload) {
  return {
    accessToken: jwt.sign(
      payload,
      process.env.ACCESS_TOKEN_SECRET,
      jwtConfig.access,
    ),
    refreshToken: jwt.sign(
      payload,
      process.env.REFRESH_TOKEN_SECRET,
      jwtConfig.refresh,
    ),
  };
}

module.exports = generateTokens;
```

в папке ws создаем файлы connection.js, upgrade.js, wsServer.js

connection.js

```
const { Message, User } = require('../../db/models');

const connections = new Map(); // in-memory DB

const connectionCb = async (socket, request, user) => {
  connections.set(user.id, { ws: socket, user });

  connections.forEach(async (connection) => {
    const { ws } = connection;
    const allUsers = [...connections.values()].map(({ user: u }) => u);
    
    const action = {
      type: 'SET_USERS',
      payload: allUsers,
    };
    ws.send(JSON.stringify(action));
  });
  const allMessages = await Message.findAll({
    include: User,
  });
  socket.send(JSON.stringify({ type: 'SET_MESSAGES', payload: allMessages }));

  socket.on('close', () => {
    connections.delete(user.id);
    connections.forEach((connection) => {
      const { ws } = connection;
      const allUsers = [...connections.values()].map(({ user: u }) => u);
      const action = {
        type: 'SET_USERS',
        payload: allUsers,
      };
      ws.send(JSON.stringify(action));
    });
  });

  socket.on('error', () => {
    connections.delete(user.id);
    connections.forEach((connection) => {
      const { ws } = connection;
      const allUsers = [...connections.values()].map(({ user: u }) => u);
      const action = {
        type: 'SET_USERS',
        payload: allUsers,
      };
      ws.send(JSON.stringify(action));
    });
  });

  socket.on('message', async (message) => {
    const actionFromFront = JSON.parse(message);
    const { type, payload } = actionFromFront;
    switch (type) {
      case 'NEW_MESSAGE':
        {
          const newMessage = await Message.create({ body: payload.text, userId: user.id });
          const newMessageWithUser = await Message.findOne({
            where: { id: newMessage.id },
            include: User,
          });
          const action = {
            type: 'ADD_MESSAGE',
            payload: newMessageWithUser,
          };
          connections.forEach((connection) => {
            const { ws } = connection;
            ws.send(JSON.stringify(action));
          });
        }
        break;

      default:
        break;
    }
  });

  //   const userId = request.session.user.id;

  //   map.set(userId, { ws: socket, user: request.session.user });

  //   function sendUsers(activeConnections) {
  //     activeConnections.forEach(({ ws }) => {
  //       ws.send(
  //         JSON.stringify({
  //           type: 'SET_USERS',
  //           payload: [...map.values()].map(({ user }) => user),
  //         }),
  //       );
  //     });
  //   }

  //   sendUsers(map);

  //     const actionFromFront = JSON.parse(message);
  //     const { type, payload } = actionFromFront;
  //     switch (type) {
  //       case SEND_MESSAGE:
  //         Message.create({ text: payload, authorId: userId }).then(async (newMessage) => {
  //           const newMessageWithAuthor = await Message.findOne({
  //             where: { id: newMessage.id },
  //             include: User,
  //           });
  //           map.forEach(({ ws }) => {
  //             ws.send(
  //               JSON.stringify({
  //                 type: ADD_MESSAGE,
  //                 payload: newMessageWithAuthor,
  //               }),
  //             );
  //           });
  //         });
  //         break;

  //       case DELETE_MESSAGE:
  //         Message.findOne({ where: { id: payload } }).then(async (targetMessage) => {
  //           if (targetMessage.authorId !== userId) return;
  //           await Message.destroy({ where: { id: payload } });
  //           map.forEach(({ ws }) => {
  //             ws.send(
  //               JSON.stringify({
  //                 type: HIDE_MESSAGE,
  //                 payload,
  //               }),
  //             );
  //           });
  //         });
  //         break;

  //       default:
  //         break;
  //     }
  //     console.log(`Received message ${message} from user ${userId}`);
  //   });

  //   socket.on('close', () => {
  //     map.delete(userId);
  //     sendUsers(map);
  //   });
};

module.exports = connectionCb;

```

upgrade.js

```
const cookieParser = require('cookie-parser');
const jwt = require('jsonwebtoken');
const wsServer = require('./wsServer');
require('dotenv').config();

const upgradeCb = (request, socket, head) => {
  socket.on('error', (err) => {
    console.log('Socket error:', err);
  });

  console.log('Parsing token from request...');
  cookieParser()(request, {}, () => {
    try {
      const { refreshToken } = request.cookies;
      const { user } = jwt.verify(
        refreshToken,
        process.env.REFRESH_TOKEN_SECRET,
      );
      if (!user) throw new Error('Нет юзера в куки');

      console.log('JWT is parsed!');

      socket.removeListener('error', (err) => {
        console.log('Socket error:', err);
      });

      wsServer.handleUpgrade(request, socket, head, (ws) => {
        wsServer.emit('connection', ws, request, user);
      });
    } catch (error) {
      socket.write('HTTP/1.1 401 Unauthorized\n\n');
      socket.destroy();
    }
  });
};

module.exports = upgradeCb;
```

wsServer.js

```
const { WebSocketServer } = require('ws');

const wsServer = new WebSocketServer({
    clientTracking: false,
    noServer: true,
  });

  module.exports = wsServer;
```

в файле app.js

```
const express = require('express');
const morgan = require('morgan');
const {createServer} = require('http');
const cookieParser = require('cookie-parser');
const userRouter = require('./router/userRouter');
const postRouter = require('./router/postRouter');
const authRouter = require('./router/authRouter');
const tokenRouter = require('./router/tokensRouter');
const likeRouter = require('./router/likeRouter');
const commentRouter = require('./router/commentRouter');
const upgradeCb = require('./ws/upgrade');
const wsServer = require('./ws/wsServer');
const connection = require('./ws/connection');

const app = express();

app.use(morgan('dev'));
app.use(express.json());
app.use(cookieParser());
app.use(express.urlencoded({ extended: true }));

app.use('/api/users', userRouter);
app.use('/api/posts', postRouter);
app.use('/api/auth', authRouter);
app.use('/api/tokens', tokenRouter);
app.use('/api/likes', likeRouter);
app.use('/api/comments', commentRouter)

const server = createServer(app);

server.on('upgrade', upgradeCb)
wsServer.on('connection', connection)

module.exports = server;

```

server.js

```
const app = require('./app')
require('dotenv').config();

const PORT = process.env.PORT;

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

# Заключение

Разделяем терминал на два в одном пишем

```
cd client/
```

а в другом пишем

```
cd server/
```

и там и там пишем

```
npm run dev
```

и проверяем работоспособность!

Общая структура проекта должна выглядить примерно так

```
client--
       |
       > node_moduels
       |
       > public
       |
       > src
            |
            > assets
            |
            > components
                       |
                       > hoc
                           |
                           > ProtectedRoute.jsx
                       |
                       > pages
                             |
                             > AccountPage.jsx
                             |
                             > ChatPage.jsx
                             |
                             > MainPage.jsx
                             |
                             > PostPage.jsx
                             |
                             > SignInPage.jsx
                             |
                             > SignUpPage.jsx
                             |
                             > WelcomePage.jsx
                             |
                             
                       |
                       > ui
                          |
                          > ChatComponent.jsx
                          |
                          > ChatInput.jsx
                          |
                          > ChatList.jsx
                          |
                          > ChatMessage.jsx
                          |
                          > CommentsCard.jsx
                          |
                          > Modal.jsx
                          |
                          > NavBar.jsx
                          |
                          > PostCard.jsx
                          |
                          > Spinner.jsx
            |
            Layout.jsx
         |
         App.jsx
         |
         axiosInstance.js
         |
         main.jsx
```

```
server--
       |
       > config
              |
              database.js
       |
       > db
          |
          > migrations
          > models
          > seedrs
       |
       > src
           |
           > configs
                   |
                   > cookieConfig.js
                   |
                   > jwtConfig.js
           |
           > middlewares
                       |
                       > verifyTokens.js
           |
           > router
                  |
                  > authRouter.js
                  |
                  > postRouter.js
                  |
                  > tokensRouter.js
                  |
                  userRouter.js
           |
           > utlis
                 |
                 > generateTokens.js
           |
           > ws
                 |
                 > connection.js
                 |
                 > upgrade.js
                 |
                 > wsServer.js
           |
           app.js
           |
           server.js
       |
       .env
       |
       .env.example
       |
       .gitignore
       |
       .sequelizerc

```
