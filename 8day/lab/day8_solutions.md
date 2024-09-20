# Lab: Progressive Authentication and Role-Based Access with FastAPI

- [Lab: Progressive Authentication and Role-Based Access with FastAPI](#lab-progressive-authentication-and-role-based-access-with-fastapi)
  - [Objectives](#objectives)
  - [Theoretical Concepts](#theoretical-concepts)
    - [What is OAuth2?](#what-is-oauth2)
      - [Why OAuth2?](#why-oauth2)
    - [What are Bearer Tokens?](#what-are-bearer-tokens)
      - [Why Use Bearer Tokens?](#why-use-bearer-tokens)
    - [What is JWT (JSON Web Token)?](#what-is-jwt-json-web-token)
      - [Why JWT for Authentication?](#why-jwt-for-authentication)
      - [How Does JWT Work in Authentication?](#how-does-jwt-work-in-authentication)
    - [Comparison: HTTP Basic Authentication vs. OAuth2 with JWT](#comparison-http-basic-authentication-vs-oauth2-with-jwt)
  - [Part 1: Simple Prediction Endpoint with Basic Authentication](#part-1-simple-prediction-endpoint-with-basic-authentication)
    - [Task 1: Set Up the Prediction Endpoint](#task-1-set-up-the-prediction-endpoint)
    - [Task 2: Add Basic Authentication to the Prediction Endpoint](#task-2-add-basic-authentication-to-the-prediction-endpoint)
  - [Part 2: Introduce Role-Based Access](#part-2-introduce-role-based-access)
    - [Task 1: Create a Top Secret Endpoint](#task-1-create-a-top-secret-endpoint)
  - [Part 3: OAuth2 with Password and Bearer Token Authentication](#part-3-oauth2-with-password-and-bearer-token-authentication)
    - [Introduction](#introduction)
    - [Flow of OAuth2 Password Authentication](#flow-of-oauth2-password-authentication)
    - [Client Credentials (Not Needed in Password Flow)](#client-credentials-not-needed-in-password-flow)
    - [Task 1: OAuth2 Setup: Implementing Username and Password Authentication](#task-1-oauth2-setup-implementing-username-and-password-authentication)
  - [`BONUS` Part 4: OAuth2 with JWT Tokens](#bonus-part-4-oauth2-with-jwt-tokens)
    - [Introduction](#introduction-1)
    - [Why JWT?](#why-jwt)
    - [Flow of OAuth2 with JWT Authentication](#flow-of-oauth2-with-jwt-authentication)
    - [What’s Inside a JWT?](#whats-inside-a-jwt)
    - [Task 1: Implementing JWT Token Authentication](#task-1-implementing-jwt-token-authentication)
    - [Task 2: Securing the Admin-Only `/secret` Endpoint](#task-2-securing-the-admin-only-secret-endpoint)
  - [Conclusion](#conclusion)
  - [Useful Links](#useful-links)

## Objectives

- Implement basic authentication for a prediction API endpoint.
- Gradually introduce more advanced authentication, including role-based access control.
- Secure a "top secret" admin-only endpoint.

## Theoretical Concepts

Before we dive into the hands-on sections of this lab, let's take some time to understand the core theoretical concepts behind **OAuth2**, **Bearer Tokens**, and **JWT (JSON Web Tokens)**. These concepts form the foundation of secure authentication and authorization in modern APIs.

---

### What is OAuth2?

**OAuth2** (Open Authorization) is an open standard for token-based authentication and authorization on the internet. It allows third-party services (like mobile apps or web applications) to access user information without exposing the user's credentials.

OAuth2 defines several types of "flows" to accommodate different use cases:

- **Password Flow**: The user provides their username and password directly to the server, and the server issues an access token.
- **Client Credentials Flow**: Used when a client (e.g., an app) needs to authenticate without user interaction.
- **Authorization Code Flow**: Commonly used for web applications, where a third-party service like Google or Facebook acts as an identity provider.

In this lab, we will focus on the **Password Flow**, where the user directly submits their username and password to the server, and in return, the server provides an access token.

#### Why OAuth2?

- **Separation of Concerns**: OAuth2 separates the process of authentication from authorization. For instance, users can authenticate once and authorize multiple services with just a token.
- **Security**: OAuth2 allows us to use tokens (instead of passwords) to access resources, minimizing the exposure of sensitive information.
- **Scalability**: The token-based approach of OAuth2 is highly scalable, making it a great choice for APIs and distributed systems.

---

### What are Bearer Tokens?

A **Bearer Token** is a type of access token that is used in OAuth2 to authenticate API requests. The token is included in the **Authorization** header of HTTP requests.

Bearer tokens are simple but powerful: they are **opaque** (you don't need to know what's inside) and treated as a credential that grants access to specific resources. Whoever holds the token (the "bearer") has access to those resources.

#### Why Use Bearer Tokens?

- **Security**: Tokens are short-lived and can be revoked, providing better security than using passwords repeatedly.
- **Efficiency**: After initial authentication, the token is used in place of the user's credentials, reducing the need to re-authenticate for each request.
- **Flexibility**: Bearer tokens are easily implemented across different platforms and services, making them a flexible option for authentication.

---

### What is JWT (JSON Web Token)?

**JWT (JSON Web Token)** is a compact, URL-safe way to represent claims between two parties. These claims are encoded in a JSON object and digitally signed. JWTs are used for authentication in web applications and APIs because they are self-contained and portable.

A JWT consists of three parts:

1. **Header**: Contains the type of token (JWT) and the signing algorithm (e.g., HS256).
2. **Payload**: Contains claims or assertions. For example, a user’s identity (`sub`), expiration time (`exp`), and custom data.
3. **Signature**: Used to verify the token and ensure it hasn’t been altered after it was issued.

#### Why JWT for Authentication?

- **Self-contained**: JWTs are self-contained, meaning the token itself holds all the necessary information (user identity, roles, expiration). This reduces the need for server-side session storage.
- **Stateless**: Since JWTs contain all relevant information, they are well-suited for stateless APIs, making them scalable.
- **Security**: JWTs are signed, ensuring the integrity of the token and preventing tampering.

#### How Does JWT Work in Authentication?

1. **Login**: The user submits their credentials (username and password) to the server.
2. **Token Issuance**: The server verifies the credentials and issues a signed JWT, containing user details and claims (such as roles and expiration).
3. **Token Usage**: The client stores the JWT and sends it in the **Authorization** header of subsequent API requests.
4. **Token Verification**: The server decodes and verifies the JWT’s signature, checking its validity and expiration before granting access.

---

### Comparison: HTTP Basic Authentication vs. OAuth2 with JWT

| Feature                         | HTTP Basic Authentication                     | OAuth2 with JWT                                 |
|----------------------------------|-----------------------------------------------|-------------------------------------------------|
| **Transmission**                 | Username and password sent with every request | Token sent in the Authorization header          |
| **Security**                     | Low (credentials sent repeatedly)             | High (token signed and can expire)              |
| **Scalability**                  | Low (server needs to manage credentials)      | High (stateless, no server-side session storage)|
| **Token Expiration**             | No                                            | Yes (tokens expire after a set duration)        |
| **Bearer Token Usage**           | No                                            | Yes                                             |
| **Role-based Access**            | Limited                                       | Easily integrated with JWT claims               |

## Part 1: Simple Prediction Endpoint with Basic Authentication

### Task 1: Set Up the Prediction Endpoint

**Objective**: Build a `/predict` endpoint that takes input data, runs the pre-trained model, and returns the predictions.

**Instructions**:

- Create a **FastAPI** project and expose a simple `/predict` endpoint that returns some text.
- No authentication yet; just focus on ensuring that the API can take input and return output.

**Solution**:

```python
from fastapi import FastAPI

app = FastAPI()

@app.post("/predict")
def predict(data: dict):
    return {"prediction": "Sample prediction"}
```

### Task 2: Add Basic Authentication to the Prediction Endpoint

Objective: Secure the `/predict` endpoint using HTTP Basic Authentication.

**Instructions:**

- Generate a fake user db with hardcoded usernames and passwords for the users.
- Add a dummy `hash` function that takes as input the password and produces `hashed{password}`. This is just so we can mimic hashing.
- Use HTTP Basic Authentication to protect the `/predict` endpoint.
- Only authenticated users should be able to access the `/predict` endpoint.
- Check your endpoint both in UI and with curl. When using curl checkout the `-u username:password` flag.
  
**Solution:**

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBasic, HTTPBasicCredentials

def hash_password(password):
    return "hashed" + password

# Basic username/password database
users_db = {
    "user": {"username": "user", "password": hash_password("userpass"), "role": "user"},
}

app = FastAPI()
security = HTTPBasic()

def authenticate_user(credentials: HTTPBasicCredentials = Depends(security)):
    user = users_db.get(credentials.username)
    if not user or user["password"] !=  hash_password(credentials.password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Basic"},
        )
    return {"username": credentials.username}

@app.post("/predict")
def predict(data: dict, credentials: HTTPBasicCredentials = Depends(authenticate_user)):
    return {"prediction": "Sample prediction"}
```

## Part 2: Introduce Role-Based Access

### Task 1: Create a Top Secret Endpoint

Objective: Add a new endpoint, `/secret`, which should only be accessible by admin users.

**Instructions:**

- Create a `secret` endpoint that returns `Admin Only: This is the secret data!`
- Add another role, `admin`, with access to this top-secret endpoint.
- Users with the `user` role should not have access to this endpoint.
- Both `user` and `admin` should have access to prediction.

**Solution:**

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBasic, HTTPBasicCredentials

def hash_password(password):
    return "hashed" + password

# Basic username/password database
users_db = {
    "user": {"username": "user", "password": hash_password("userpass"), "role": "user"},
    "admin": {"username": "admin", "password": hash_password("adminpass"), "role": "admin"},
}

app = FastAPI()
security = HTTPBasic()

def authenticate_user(credentials: HTTPBasicCredentials = Depends(security)):
    user = users_db.get(credentials.username)
    if not user or user["password"] != hash_password(credentials.password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Basic"},
        )
    return {"username": credentials.username, "role": user['role']}

@app.post("/predict")
def predict(data: dict, credentials: HTTPBasicCredentials = Depends(authenticate_user)):
    return {"prediction": "Sample prediction"}

@app.get("/secret")
def secret_endpoint(credentials: HTTPBasicCredentials = Depends(authenticate_user)):
    if credentials["role"] != "admin":
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Not enough permissions")
    return {"message": "Admin Only: This is the secret data!"}
```

## Part 3: OAuth2 with Password and Bearer Token Authentication

### Introduction

In this part of the lab, we will focus on OAuth2 with password-based authentication.

With OAuth2 and the "password flow," users will log in using their username and password. The server will verify the credentials and return a token, which the client can use to access secured resources. This token acts as proof that the user has been authenticated and is allowed to access the endpoints

By upgrading to OAuth2:

- The user's credentials (username and password) are only sent once during login.
- The token is used instead of credentials, reducing the exposure of sensitive information.
- Tokens can be set to expire, enhancing security.

### Flow of OAuth2 Password Authentication

- **Login (Authorize)**: The client (user) sends their username and password to a specific `/token` endpoint. The server verifies the credentials.
- **Token Generation**: If the credentials are correct, the server generates a Bearer Token (an access token), which is sent back to the client.
- **Token Storage**: The client stores this token.
- **Authenticated Requests**: For subsequent requests, the client includes this token in the Authorization header of the request. This proves that the user has already authenticated.

### Client Credentials (Not Needed in Password Flow)

In the OAuth2 password flow, the client (user) provides their own username and password. Client credentials like client_id and client_secret are not typically used in this flow. These credentials are more relevant in Client Credentials Flow, where the client (an application or service) authenticates itself, rather than a user.

For now, the client credentials (client_id, client_secret) are not required for this part of the lab, as we are focused on user-based authentication (OAuth2 with password flow).

### Task 1: OAuth2 Setup: Implementing Username and Password Authentication

Let’s walk through the steps of upgrading our authentication to use OAuth2 with password flow.

- Step 1: Create the `/token` Endpoint for Logging In
  - The first step is to create a `/token` endpoint where the user can send their username and password to log in.
  - This endpoint will:
    - Accept the user’s credentials via form data.
    - Verify the credentials.
    - Return an access token if the credentials are valid. For now, for the access token you can just return the username.
- Step 2: Securing the `/predict` Endpoint with the Bearer Token
  - Now that we have a way to log in and get a token, we can secure our existing `/predict` endpoint so that it only accepts requests with a valid token. The real verification will be done in the next part. This part is just a basic setup.
- Step 3: Access the `/predict` Endpoint with curl.

**Solutions**:

```python
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

def hash_password(password):
    return "hashed" + password

# Basic username/password database
users_db = {
    "user": {"username": "user", "password": hash_password("userpass"), "role": "user"},
    "admin": {"username": "admin", "password": hash_password("adminpass"), "role": "admin"},
}

app = FastAPI()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

# Token generation endpoint
@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = users_db.get(form_data.username)
    if not user or user["password"] != hash_password(form_data.password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    # In a real application, you would generate a proper JWT or token here
    return {"access_token": user["username"], "token_type": "bearer"}

@app.post("/predict")
async def predict(data: dict, token: str = Depends(oauth2_scheme)):
    # TODO: verify (decode) the token --> to be done in part4
    return {"prediction": "Sample prediction", "token": token}
```

```bash
curl -X 'POST' \
  'http://localhost:8000/predict' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer admin' \
  -H 'Content-Type: application/json' \
  -d '{}'
```

## `BONUS` Part 4: OAuth2 with JWT Tokens

### Introduction

In this part, we will take the previous implementation of OAuth2 and enhance it by replacing the simple access token with a JWT (JSON Web Token). JWTs are widely used because they allow us to securely encode and verify information (like user authentication) in a compact, self-contained way.

Using JWTs offers the following advantages:

- **Security**: The token is signed, ensuring it hasn't been tampered with.
- **Efficiency**: Tokens are self-contained, meaning the server doesn’t need to store session information. The token itself holds all necessary claims (e.g., user identity, expiration time).
- **Scalability**: JWT-based authentication systems are easier to scale because no server-side session storage is required.

### Why JWT?

While the previous implementation used a basic access token (the username), JWTs encode additional information (claims) like the user’s identity, token expiration, and any roles or permissions they may have. The JWT can be cryptographically signed to ensure it hasn't been altered after it was issued.

In a JWT-based system:

- Users authenticate via the /token endpoint using their username and password.
- A JWT token is returned and stored on the client side.
- The token is included in the Authorization header of subsequent requests.
- The server validates the token by decoding and verifying it, ensuring it's still valid and untampered.

### Flow of OAuth2 with JWT Authentication

- Login (Authorize): The client (user) sends their username and password to a /token endpoint. The server verifies the credentials and returns a signed JWT.
- Token Generation: The JWT contains the user’s identity, an expiration time, and other claims. The token is cryptographically signed by the server.
- Token Storage: The client stores the JWT securely.
- Authenticated Requests: For subsequent requests, the client sends the JWT in the Authorization header. The server verifies the token’s signature and validity.
  
### What’s Inside a JWT?

A JWT has three parts:

- Header: Contains the algorithm used for signing (e.g., HS256).
- Payload: Contains claims such as the user identity (sub), expiration time (exp), and custom data.
- Signature: Verifies that the token hasn't been altered.

### Task 1: Implementing JWT Token Authentication

We will modify our authentication system to issue and verify JWT tokens. Let's walk through the steps of implementing JWT-based OAuth2.

- Step 1: Install the Required Libraries
You will need the `pyjwt` and `passlib` libraries for working with JWT tokens and hashing passwords.

```
pip install pyjwt passlib[bcrypt]
```

- Step 2: Update the `/token` Endpoint to Issue JWT Tokens. Feel free to use `CryptContext` from `passlib` to generate real (password) hashes.
  - You can set the `exp` field in payload to have automatic expiration checks during decode phase.
- Step 3: Update the `/predict` Endpoint to Use JWT Tokens (i.e., decode and verify). The client must send the token in the Authorization header. Additionally, when returning the prediction also return the username of the logged in user (decoded from the token).
  - Make sure to decode the token.
  - Verify if the information inside of it is correct, i.e. allows the user to access content.
- Step 4: Test out the `/token` Endpoint and initiate requests from UI and curl.

**Solution:**

```python
from datetime import datetime, timedelta, timezone
from typing import Union
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from passlib.context import CryptContext
import jwt
from jwt import PyJWTError

# JWT settings
SECRET_KEY = "your_secret_key"  # Replace with your own secret key
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# Password hashing context
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

app = FastAPI()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

# User database
users_db = {
    "admin": {"username": "admin", "password": pwd_context.hash("adminpass"), "role": "admin"},
    "user": {"username": "user", "password": pwd_context.hash("userpass"), "role": "user"},
}


# Helper functions
def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)


def authenticate_user(username: str, password: str):
    user = users_db.get(username)
    if not user or not verify_password(password, user["password"]):
        return False
    return user


def create_access_token(data: dict, expires_delta: Union[timedelta, None] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.now(timezone.utc) + expires_delta
    else:
        expire = datetime.now(timezone.utc) + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user["username"]}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}


def decode_token(token: str):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid token"
            )
        return username
    except PyJWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid token"
        )


@app.post("/predict")
async def predict(data: dict, token: str = Depends(oauth2_scheme)):
    username = decode_token(token)
    # TODO: Do something with the informatino from decoded token.
    return {"prediction": "Sample prediction", "user": username}
```

```bash
curl -X 'POST' \
  'http://localhost:8000/predict' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyIiwiZXhwIjoxNzI2NTcyNjY3fQ.3Fpj72ohhs7F0m3xhk4m_mhO6vwNFQ9vjlw64iFZ2tA' \
  -H 'Content-Type: application/json' \
  -d '{}'
```

### Task 2: Securing the Admin-Only `/secret` Endpoint

Instructions:

- Update the JWT payload to include the user's role when creating the access token.
- Secure the `/secret` endpoint by checking if the role in the token is "admin".

**Solution:**

- Step 1: Modify the `create_access_token` function to include the user's role in the JWT token. When calling this function, we’ll pass the user’s role along with the username.

```python
def create_access_token(data: dict, role: str, expires_delta: Union[timedelta, None] = None):
    to_encode = data.copy()
    to_encode.update({"role": role})  # Add role to the payload
    if expires_delta:
        expire = datetime.now(timezone.utc) + expires_delta
    else:
        expire = datetime.now(timezone.utc) + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt
```

- Step 2: Update the `/token` endpoint to include the user’s role in the token.
Now, each generated token will contain both the username (sub) and the role (role).

```python
@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user["username"]}, role=user["role"], expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}
```

- Step 3: Secure the `/secret` Endpoint for Admin Access. First, we’ll decode the token and extract the role. Then, we’ll verify whether the role is "admin" before allowing access to the endpoint.

```python
@app.get("/secret")
async def secret_endpoint(token: str = Depends(oauth2_scheme)):
    payload = decode_token(token)
    role = payload.get("role")
    if role != "admin":
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Not enough permissions")
    return {"message": "Admin Only: This is the secret data!"}
```

**NOTE:** A key concept to note is the `secret-key` during the encode and decode phase. Because this information is private, only our server can generate and decode such tokens.

## Conclusion

In this lab, we’ve progressed through different levels of securing an API using FastAPI, starting with basic authentication and culminating in OAuth2 with JWT tokens. Each approach brings different levels of security, scalability, and flexibility, but JWT tokens, in particular, provide the most robust and scalable solution for modern API development.

You now have a solid understanding of:

- The differences between HTTP Basic Authentication and OAuth2 with Bearer Tokens.
- How OAuth2 Password Flow works to issue and manage access tokens.
- How to use JWT tokens to securely manage authentication and authorization in your FastAPI applications.
- How to implement role-based access control to secure specific API endpoints.

## Useful Links

Here are some additional resources to help deepen your understanding of the concepts covered in this lab:

- [OAuth2 Specification](https://tools.ietf.org/html/rfc6749)
- [JWT Introduction (jwt.io)](https://jwt.io/introduction)
- [FastAPI Security - OAuth2](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/)
- [PyJWT Documentation](https://pyjwt.readthedocs.io/en/latest/)
- [Passlib Password Hashing](https://passlib.readthedocs.io/en/stable/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Everything Curl: Passwords](https://everything.curl.dev/cmdline/passwords.html)
