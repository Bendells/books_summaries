
### Chapter 4: JWT security ###

JWT: **Json Web Token**, also pronounced **JOT**. It was proposed by the **Internet Engineering task force (IETF)**. It is a Json object that contains multiple claims and is signed by **Message authentication Code (MAC)**. You usually want an external entity to generate and authenticate JWT for the application.

Dependency needed: 

```shell
./mvnw quarkus:add-extension -Dextensions=smallrye-jwt,smallrye-jwt-build
```

smallrye-jwt is an implementation of Eclipse Microprofile JWT RBAC security specifications. It allows us to authenticate jwt tokens that are passed through by the user. Also makes it available for injection

smallrye-jwt-build allows us to generate JWT tokens.

Signed tokens, we need to be using a private and public token for asymetric authentication.

Authorization service holds the private key and distribute the JWT token. Authenticated by the public key on the application. For this chapter, application will do both.

#### Setup ####
- Create a **jwt** directory under resources, navigate to it and run:

```shell
openssl genrsa -out rsa-private-key.pem 2048
```
- Convert the private key to PKCS#8 format to make it compatible with SmallRye JWT:
```shell
openssl pkcs8 -topk8 -nocrypt -inform pem -in rsa-private-key.pem -outform pem -out private-key.pem
```
- Then generate the public key:
```shell
openssl rsa -pubout -in rsa-private-key.pem -out public-key.pem
```

The above allows us to create the private key used to generate tokens and the public keys used to validate them during authentication.

To allow the application to use them, we need to update the configuration as such:


```shell
smallrye.jwt.sign.key.location=jwt/private-key.pem
mp.jwt.verify.publickey.location=jwt/public-key.pem
mp.jwt.verify.issuer=https://example.com/issuer
```

Those define the location of the sing key (private key), public key and the authorirty (in this case example.com)

#### Implementing the authentication service and login interface ####

- Add a match function to the **UserService** class 
- Add an **AuthRequest** record
- Add an **AuthService** class to handle authorization
- Add an **AuthResource** class to handle the login API

JsonWebToken allows us to have information about the current request session

### Chapter 5: Testing your backend ###

Testing with Quarkus is similair to testing with JUnit 5 with some minor difference. Quarkus runs the application on the background when started so it is easy to test different layers. Dependecy injection is still possible but with Quarkus specific instructions

Because we use JWT, we need to add testing security for them, this is available through the quarkus-test-security-jwt.

```Shell
./mvnw quarkus:add-extension -Dextensions=io.quarkus:quarkus-test-security-jwt
```
Note to Self: Above didn't work so had to look at the Maven repo and used this 

```Shell
./mvnw quarkus:add-extension -Dextensions=io.quarkus:quarkus-test-security-jwt:3.13.1
```

- It depends on `quarkus-test-security` 
- Adding `<scope>test</scope>` allows the extension to only be usable within the tests

#### Testing the Task Manager ####

Unit testing all the components in the application will be tedious and would require complex test and mocking [ Slim's Note: comprehensive (different from full) unit testing should be the norm and not the exception, spending time on setting up mocking and having a high coverage will pay dividends in the future ]. The book goes by implementing an integration test at each layer instead.

To add mock data, we point the `test` variable to the `dev` sql file used when the dev server is spun up:
```Shell
%test.quarkus.hibernate-orm.sql-load-script=import-dev.sql
```

- Create a new AuthResourceTest file (easy to create through opening AuthResource and right clicking to Test)
- Use RestAssured library (rest request testing)

AuthResourceTest:

```Java
package com.example.fullstack.auth;  
  
import io.quarkus.test.junit.QuarkusTest;  
import io.restassured.http.ContentType;  
import org.hamcrest.Matchers;  
import org.junit.jupiter.api.Test;  
  
import static io.restassured.RestAssured.given;  
  
@QuarkusTest  
class AuthResourceTest {  
    @Test  
    void loginValidCredentials() {  
        given()  
                .body("{\"name\":\"admin\",\"password\":\"quarkus\"}")  
                .contentType(ContentType.JSON)  
                .when().post("/api/v1/auth/login")  
                .then()  
                .statusCode(200)  
                .body(Matchers.not(Matchers.emptyString()));  
    }  
    @Test  
    void loginInvalidCredentials() {  
        given()  
                .body("{\"name\":\"admin\",\"password\":\"not-quarkus\"}")  
                .contentType(ContentType.JSON)  
                .when().post("/api/v1/auth/login")  
                .then()  
                .statusCode(401);  
    }  
}
```

2 tests to validate login credentials, a positive one with a `200` return code and an invalid one with an expected `401` return code.