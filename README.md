# JwtUtil
*A library and CLI application for creating, reading, and verifying JSON Web Tokens (JWTs)*

## Building

Build with [CMake](https://cmake.org/) as normal:
```
cmake -B build
cmake --build build
```

## Command-line Usage

Run `build/JwtUtil --help` or `build/JwtUtil [subcommand] --help`


## Command-line Examples

Generating an HMAC key:
```sh
$ build/JwtUtil key oct HS256 -l 32 > key_hs256.json
```

Generating a 2048-bit (256-byte) RSA key pair:
```sh
$ build/JwtUtil key RSA RS256 -l 256 > key_rs256.json
```

Generating a P-256 elliptic curve key pair:
```sh
$ build/JwtUtil key EC ES256 -c P-256 > key_es256.json
```

Generating a signed token:
```
$ build/JwtUtil create -k key_hs256.json --issuer myapp --expires-in 300 HS256
```

Generating an encrypted token:
```
$ build/JwtUtil create -k key_rs256.json --issuer myapp --expires-in 300 RS256
```

Verifying a token:
```
$ build/JwtUtil verify -k key_hs256.json --issuer myapp "<token>"
```

Verifying a token, then printing its payload as JSON:
```
$ build/JwtUtil verify -o payload -k key_hs256.json --issuer myapp "<token>"
```


## Including the Library

#### Option 1: Fetch with [CPM.cmake](https://github.com/cpm-cmake/CPM.cmake) 

First, download CPM.cmake:
```sh
$ mkdir -p cmake
$ wget -O cmake/CPM.cmake https://github.com/cpm-cmake/CPM.cmake/releases/latest/download/get_cpm.cmake
```

Then, configure your CMakeLists.txt as follows:
```cmake
include(cmake/CPM.cmake)
CPMAddPackage("gh:jwall64/jwtutil@0.1.0")

add_executable(myapp main.c)
target_link_libraries(myapp JwtLib)
```

#### Option 2: Add as a Git Submodule

First, add the submodule
```sh
$ git submodule add https://github.com/jwall64/JwtUtilC.git external/JwtUtil 
```

Then, configure your CMakeLists.txt as follows:
```cmake
add_subdirectory(external/JwtUtil)

add_executable(myapp main.c)
target_link_libraries(myapp JwtLib)
```


## Library Usage Examples

Reading a JWK from a file:
```c

JwtReader keyReader;
JwtJsonElement keyJson;
JwtKey key;

if(jwtReaderCreateForFile(&keyReader, "key.json") < 0) {
    return EXIT_FAILURE;
}

if(jwtReadJsonReader(&keyJson, keyReader) < 0) {
    jwtReaderClose(keyReader);
    return EXIT_FAILURE;
}
jwtReaderClose(keyReader);

if(keyJson.type != JWT_JSON_ELEMENT_TYPE_OBJECT || jwtKeyParse(&key, keyJson.object) < 0) {
    jwtJsonElementDestroy(&keyJson);
    return EXIT_FAILURE;
}

jwtJsonElementDestroy(&keyJson);

// Do things with the key

jwtKeyDestroy(&key);

```


Creating a signed JWT:
```c

JwtKey key; // See above
JwtAlgorithm alg = JWT_ALGORITHM_HS256;
JwtJsonObject payload;
JwtString token;

if(jwtJsonObjectCreate(&payload) < 0) {
    return EXIT_FAILURE;
}

jwtJsonObjectSetString(&payload, JWT_CLAIM_ISSUER, "myapp");
jwtJsonObjectSetUint(&payload, JWT_CLAIM_EXPIRATION, 0); // Some timestamp

if(jwtCreateSignedToken(&payload, &key, alg, &token) < 0) {
    jwtJsonObjectDestroy(&payload);
    return EXIT_FAILURE;
}
jwtJsonObjectDestroy(&payload);

// Do things with the token

jwtStringDestroy(&token);

```


Creating an encrypted JWT:
```c

JwtKey key; // See above
JwtAlgorithm alg = JWT_ALGORITHM_A128KW;
JwtCryptAlgorithm enc = JWT_CRYPT_ALGORITHM_A128CBC_HS256;
JwtJsonObject payload;
JwtString token;

if(jwtJsonObjectCreate(&payload) < 0) {
    return EXIT_FAILURE;
}

jwtJsonObjectSetString(&payload, JWT_CLAIM_ISSUER, "myapp");
jwtJsonObjectSetUint(&payload, JWT_CLAIM_EXPIRATION, 0); // Some timestamp

if(jwtCreateEncryptedToken(&payload, &key, alg, enc, &token) < 0) {
    jwtJsonObjectDestroy(&payload);
    return EXIT_FAILURE;
}
jwtJsonObjectDestroy(&payload);

// Do things with the token

jwtStringDestroy(&token);

```


Verifying a JWT:
```c

JwtKey key; // See above
const char* tokenCStr; // From user
JwtString token = { .length = strlen(tokenCStr), .data = tokenCStr };
JwtParsedToken parsed;

if(jwtVerifyToken(token, &key, &parsed, 0) < 0) {
    return EXIT_FAILURE;
}

JwtString iss = jwtJsonObjectGetString(&parsed.payload, JWT_CLAIM_ISSUER);

// Do things with the parsed token

jwtParsedTokenDestroy(&parsed);

```

