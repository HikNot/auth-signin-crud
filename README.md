# auth-signin-crud

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
npm i reac-bootstrap bootstrap react-router-dom
```

// Шаг 3 (Client)

заходим в папку client

```
cd client/
```

в клиенте создаем папку src -> components -> pages, ui, hoc и файл Layout.jsx

в hoc создаем файл ProtectedRoute.jsx

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

export default function Layout({ user, handleLogout }) {
  return (
    <>
      <NavBar user={user} handleLogout={handleLogout} />
      <Outlet />
    </>
  );
}

```

// Шаг 4 (Client)

создаем MainPage.jsx  и пишем

```
import React, { useEffect, useState } from 'react';
import axiosInstance from '../../axiosInstance';
import PostCard from '../ui/PostCard';
import Container from 'react-bootstrap/Container';
import Row from 'react-bootstrap/Row';
import Col from 'react-bootstrap/Col';

export default function MainPage({user}) {
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    axiosInstance.get('/posts').then((res) => setPosts(res.data));
  }, []);

  return (
    <>
      <Container>
        <Row className='mt-1'>
          {posts.map((post) => (
            <Col className='mt-3 mb-3' key={post.id}>
              <PostCard user={user} post={post} />
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

в ui создаем файлы Modal.jsx, NavBar.jsx, PostCard.jsx

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
        <NavLink style={{ textDecoration: 'none' }} to={'/home'}>
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

export default function PostCard({ post, deletePostHandler, setModalContent, user}) {
  
  return (
    <Card style={{ width: '18rem' }}>
      <Card.Img
        variant="top"
        src="https://w7.pngwing.com/pngs/124/1016/png-transparent-new-post-thumbnail.png"
      />
      <Card.Body>
        <Card.Title>{post.title}</Card.Title>
        <Card.Text>{post.body}</Card.Text>
      </Card.Body>
      <ListGroup className="list-group-flush">
        <ListGroup.Item>Автор: {post.User?.name}</ListGroup.Item>
      </ListGroup>
      <Card.Body>
      <Button onClick={() => deletePostHandler(post.id)} variant="danger">Удалить</Button>{' '}
      <Button onClick={() => setModalContent(post)} variant="warning">Редактировать</Button>{' '}

      </Card.Body>
    </Card>
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

function App() {
  const [user, setUser] = useState();

  useEffect(() => {
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
      });
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
          path: '/home',
          element: (
            <ProtectedRoute isAllowed={!!user} redirectPath="/home">
              <MainPage user={user}></MainPage>
            </ProtectedRoute>
          ),
        },
        {
          path: '/my-cards',
          element: (
            <ProtectedRoute isAllowed={!!user} redirectPath="/my-cards">
              <AccountPage user={user} />
            </ProtectedRoute>
          ),
        },
        {
          element: <ProtectedRoute isAllowed={!user} redirectPath='/' />,
          children: [
            {
              path: '/',
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
  <React.StrictMode>
    <App />
  </React.StrictMode>,
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
    static associate({ X }) {
      // define association here
      this.hasMany(X, { foreignKey: 'userId'});
    }
  }
```

у папки X пишем

```
class Post extends Model {
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
создаем папку src и в ней configs, middlewares, router, utils, app.js и server.js

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

в папке router создаем файлы authRouter.js, postRouter.js (назвиние может меняться в соответствии с заданием), tokensRouter.js, userRouter.js

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
в файле app.js

```
const express = require('express');
const morgan = require('morgan');
const cookieParser = require('cookie-parser');
const userRouter = require('./router/userRouter');
const postRouter = require('./router/postRouter');
const authRouter = require('./router/authRouter');
const tokenRouter = require('./router/tokensRouter');

const app = express();

app.use(morgan('dev'));
app.use(express.json());
app.use(cookieParser());
app.use(express.urlencoded({ extended: true }));

app.use('/api/users', userRouter)
app.use('/api/posts', postRouter)
app.use('/api/auth', authRouter)
app.use('/api/tokens', tokenRouter)


module.exports = app;
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
                             > MainPage.jsx
                             |
                             > SignUpPage.jsx
                             |
                             > SignInPage.jsx
                             |
                       |
                       > ui
                          |
                          > Modal.jsx
                          |
                          > NavBar.jsx
                          |
                          > PostCard.jsx
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
                 
       
