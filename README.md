# Task for Hellwet Soft

## Quick Start

Install the dependencies:

```bash
yarn install
```

Set the environment variables:

```bash
cp .env.example .env

# open .env and modify the environment variables (if needed)
```
## Table of Contents

- [Task for Hellwet Soft](#task-for-hellwet-soft)
  - [Quick Start](#quick-start)
  - [Table of Contents](#table-of-contents)
  - [Features](#features)
  - [Commands](#commands)
  - [Environment Variables](#environment-variables)
  - [Project Structure](#project-structure)
  - [API Documentation](#api-documentation)
    - [API Endpoints](#api-endpoints)
  - [Error Handling](#error-handling)
  - [Validation](#validation)
  - [Authentication](#authentication)
  - [Authorization](#authorization)
  - [Logging](#logging)
  - [Custom Mongoose Plugins](#custom-mongoose-plugins)
    - [toJSON](#tojson)

## Features

- **NoSQL database**: [MongoDB](https://www.mongodb.com) object data modeling using [Mongoose](https://mongoosejs.com)
- **Authentication and authorization**: using [passport](http://www.passportjs.org)
- **Validation**: request data validation using [Joi](https://github.com/hapijs/joi)
- **Logging**: using [winston](https://github.com/winstonjs/winston) and [morgan](https://github.com/expressjs/morgan)
- **Testing**: unit and integration tests using [Jest](https://jestjs.io)
- **Error handling**: centralized error handling mechanism
- **API documentation**: with [swagger-jsdoc](https://github.com/Surnet/swagger-jsdoc) and [swagger-ui-express](https://github.com/scottie1984/swagger-ui-express)
- **Process management**: advanced production process management using [PM2](https://pm2.keymetrics.io)
- **Dependency management**: with [Yarn](https://yarnpkg.com)
- **Environment variables**: using [dotenv](https://github.com/motdotla/dotenv) and [cross-env](https://github.com/kentcdodds/cross-env#readme)
- **Security**: set security HTTP headers using [helmet](https://helmetjs.github.io)
- **Santizing**: sanitize request data against xss and query injection
- **CORS**: Cross-Origin Resource-Sharing enabled using [cors](https://github.com/expressjs/cors)
- **Compression**: gzip compression with [compression](https://github.com/expressjs/compression)
- **CI**: continuous integration with [Travis CI](https://travis-ci.org)
- **Docker support**
- **Code coverage**: using [coveralls](https://coveralls.io)
- **Code quality**: with [Codacy](https://www.codacy.com)
- **Git hooks**: with [husky](https://github.com/typicode/husky) and [lint-staged](https://github.com/okonet/lint-staged)
- **Linting**: with [ESLint](https://eslint.org) and [Prettier](https://prettier.io)
- **Editor config**: consistent editor configuration using [EditorConfig](https://editorconfig.org)

## Commands

Running locally:

```bash
yarn dev
```

Running in production:

```bash
yarn start
```
## Environment Variables

The environment variables can be found and modified in the `.env` file. They come with these default values:

```bash
# Port number
PORT=6100

# URL of the Mongo DB
MONGODB_URL=mongodb://127.0.0.1:27017/task-hellwet-soft

# JWT
# JWT secret key
JWT_SECRET=thisisasamplesecret
# Number of minutes after which an access token expires
JWT_ACCESS_EXPIRATION_MINUTES=30
# Number of days after which a refresh token expires
JWT_REFRESH_EXPIRATION_DAYS=30

#api path
API_PATH=/api/v1

# SMTP configuration options for the email service
# For testing, you can use a fake SMTP service like Ethereal: https://ethereal.email/create
SMTP_HOST=email-server
SMTP_PORT=587
SMTP_USERNAME=email-server-username
SMTP_PASSWORD=email-server-password
EMAIL_FROM=support@yourapp.com
```
#NODE_ENV
NODE_ENV=development

## Project Structure

```
src\
 |--config\         # Environment variables and configuration related things
 |--controllers\    # Route controllers (controller layer)
 |--middlewares\    # Custom express middlewares
 |--models\         # Mongoose models (data layer)
 |--routes\         # Routes
 |--services\       # Business logic (service layer)
 |--utils\          # Utility classes and functions
 |--validations\    # Request data validation schemas
 |--app.js          # Express app
 |--index.js        # App entry point
```

## API Documentation

To view the list of available APIs and their specifications, run the server and go to `http://localhost:3000/v1/docs` in your browser. This documentation page is automatically generated using the [swagger](https://swagger.io/) definitions written as comments in the route files.

### API Endpoints

List of available routes:

**Auth routes**:\
`POST /v1/auth/register` - register\
`POST /v1/auth/login` - login\
`POST /v1/auth/logout` - logout\
`POST /v1/auth/refresh-tokens` - refresh auth tokens\
`POST /v1/auth/forgotPassword` - send reset password email\
`POST /v1/auth/verifyEmail` - verify email
`POST /v1/auth/resetPassword` - reset password\

## Error Handling

The app has a centralized error handling mechanism.

Controllers should try to catch the errors and forward them to the error handling middleware (by calling `next(error)`). For convenience, you can also wrap the controller inside the catchAsync utility wrapper, which forwards the error.

```javascript
const catchAsync = require('../utils/catchAsync');

const controller = catchAsync(async (req, res) => {
  // this error will be forwarded to the error handling middleware
  throw new Error('Something wrong happened');
});
```

The error handling middleware sends an error response, which has the following format:

```json
{
  "code": 404,
  "message": " not found"
}
```

When running in development mode, the error response also contains the error stack.

The app has a utility ApiError class to which you can attach a response code and a message, and then throw it from anywhere (catchAsync will catch it).

For example, if you are trying to get a user from the DB who is  not found, and you want to send a 404 error, the code should look something like:

```javascript
const httpStatus = require('http-status');
const ApiError = require('../utils/ApiError');
const User = require('../models/User');

const getUser = async (userId) => {
  const user = await User.findById(userId);
  if (!user) {
    throw new ApiError(httpStatus.NOT_FOUND, 'User  not found');
  }
};
```

## Validation

Request data is validated using [Joi](https://joi.dev/). Check the [documentation](https://joi.dev/api/) for more details on how to write Joi validation schemas.

The validation schemas are defined in the `src/validations` directory and are used in the routes by providing them as parameters to the `validate` middleware.

```javascript
const express = require('express');
const validate = require('../../middlewares/validate');
const userValidation = require('../../validations/user.validation');
const userController = require('../../controllers/user.controller');

const router = express.Router();

router.post('/users', validate(userValidation.createUser), userController.createUser);
```

## Authentication

To require authentication for certain routes, you can use the `auth` middleware.

```javascript
const express = require('express');
const auth = require('../../middlewares/auth');
const userController = require('../../controllers/user.controller');

const router = express.Router();

router.post('/users', auth(), userController.createUser);
```

These routes require a valid JWT access token in the Authorization request header using the Bearer schema. If the request does not contain a valid access token, an Unauthorized (401) error is thrown.

**Generating Access Tokens**:

An access token can be generated by making a successful call to the register (`POST /v1/auth/register`) or login (`POST /v1/auth/login`) endpoints. The response of these endpoints also contains refresh tokens (explained below).

An access token is valid for 30 minutes. You can modify this expiration time by changing the `JWT_ACCESS_EXPIRATION_MINUTES` environment variable in the .env file.

**Refreshing Access Tokens**:

After the access token expires, a new access token can be generated, by making a call to the refresh token endpoint (`POST /v1/auth/refresh-tokens`) and sending along a valid refresh token in the request body. This call returns a new access token and a new refresh token.

A refresh token is valid for 30 days. You can modify this expiration time by changing the `JWT_REFRESH_EXPIRATION_DAYS` environment variable in the .env file.

## Authorization

The `auth` middleware can also be used to require certain rights/permissions to access a route.

```javascript
const express = require('express');
const auth = require('../../middlewares/auth');
const userController = require('../../controllers/user.controller');

const router = express.Router();

router.post('/users', auth('manageUsers'), userController.createUser);
```

In the example above, an authenticated user can access this route only if that user has the `manageUsers` permission.

The permissions are role-based. You can view the permissions/rights of each role in the `src/config/roles.js` file.

If the user making the request does not have the required permissions to access this route, a Forbidden (403) error is thrown.

## Logging

Import the logger from `src/config/logger.js`. It is using the [Winston](https://github.com/winstonjs/winston) logging library.

Logging should be done according to the following severity levels (ascending order from most important to least important):

```javascript
const logger = require('<path to src>/config/logger');

logger.error('message'); // level 0
logger.warn('message'); // level 1
logger.info('message'); // level 2
logger.http('message'); // level 3
logger.verbose('message'); // level 4
logger.debug('message'); // level 5
```

In development mode, log messages of all severity levels will be printed to the console.

In production mode, only `info`, `warn`, and `error` logs will be printed to the console.\
It is up to the server (or process manager) to actually read them from the console and store them in log files.\
This app uses pm2 in production mode, which is already configured to store the logs in log files.

Note: API request information (request url, response code, timestamp, etc.) are also automatically logged (using [morgan](https://github.com/expressjs/morgan)).

## Custom Mongoose Plugins

The app also contains 2 custom mongoose plugins that you can attach to any mongoose model schema. You can find the plugins in `src/models/plugins`.

```javascript
const mongoose = require('mongoose');
const { toJSON, paginate } = require('./plugins');

const userSchema = mongoose.Schema(
  {
    /* schema definition here */
  },
  { timestamps: true }
);

userSchema.plugin(toJSON);
userSchema.plugin(paginate);

const User = mongoose.model('User', userSchema);
```

### toJSON

The toJSON plugin applies the following changes in the toJSON transform call:

- removes \_\_v, createdAt, updatedAt, and any schema path that has private: true
- replaces \_id with id