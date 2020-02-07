# signer

## Overview

SpringBoot library for automatic and secure http request's signing using Spring RestTemplate.

This library provide you a way for signing your http requests between SpringBoot microservices ensuring the identity of the client and the integrity of the message.
The message is signed with a keyed-hash message authentication code (HMAC) generated with a pre-shared secret key, this allow you to authenticate your rest API in a smart way.

## Description

The library offers two annotations for client and server respectively.

***@Sign***

This annotation must be placed on the client method inside wich the http call is made with Spring RestTemplate client.
The library will attach the necessary header to the request that contain the signature, everything happens transparently for the user.

***@Signed***

This annotation must be placed on the called rest endpoint method and notify the library that this method is signed and the http request must be authenticated.
If the signature verification process completes succesfully, the http request will be handled by the server otherwise the client will receive a 401 UNAUTHORIZED response message.
Every rest API annotated with *@Signed* annotation will be secured and will require a signed request. 

*Obviously the two annotations will trigger the verification process only for the request that contain a body to sign.
Putting the annotations on top of a method that accept requests without body will have no effect.*

The library uses the Java Mac class provided by the JDK to make the symmetric signature.
See the official Oracle documentation linked below for more details.

https://docs.oracle.com/javase/7/docs/api/javax/crypto/Mac.html

## Building and installation

Clone the project, build and install it with the following Maven command:

*mvn clean install -DskipTests*


## Configurations

### Library import

In order to use the library you have to import it on your pom.xml as showed below:

```
<dependency>
   <groupId>com.ffsec</groupId>
   <artifactId>signer</artifactId>
   <version>1.0</version>
</dependency>
```

### Library properties

Both the client and server must have the same secret key configured inside them since the signature algorithm use a symmetric key.
On client side the secret key is used to generate the signature, on server side the same key is used for the signature verification process.

The property to set is ***ffsec.signer.secret*** and must contains a randomic generated string with any length (recommended 128/256/512 bit).

```
ffsec.signer.secret=NV8UJUL81Y9F
```

It's also possible for the user to define the hashing algorithm used for the HMAC signature generation, the default is HmacSHA256 but also these algorithms are supported:

- HmacMD5
- HmacSHA1
- HmacSHA256
- HmacSHA384
- HmacSHA512

The property for the hashing algorithm is ***ffsec.signer.algorithm*** and the possible values are listed above.

*It's important to define the same algorithm on both client and server to avoid problems*

```
ffsec.signer.algorithm=HmacSHA384
```

## Coding Example


### Client side implementation

```
@RestController
public class ClientController {

    @Autowired
    RestTemplate restTemplate;

    @Sign
    @GetMapping("client")
    public ResponseEntity<String> test() {

        return restTemplate.postForEntity("http://localhost:8080/server", "the brown fox jumps over the lazy dog" , String.class);

    }
}
```

The library provides you an already instantiated RestTemplate bean that you can inject into your *@RestController* class or wherever it is needed.

*All the http calls must be executed with this instance otherwise the library does not work.*


### Server side implementation

```
@RestController
public class TestController {

    @Signed
    @PostMapping(value = "server", consumes = "text/plain")
    public ResponseEntity<String> demoSigned(@RequestBody String body) {
        return ResponseEntity.ok("OK");
    }
    
}
```

### Configuration class 

You have to import this configuration class on both client and server.

```
@SpringBootApplication
@Import(SignerConfiguration.class)
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

## Multithreading

The library is thread safe.
Using the two annotations at the same time on the same SpringBoot instance will not generate concurrency issues.

Since the RestTemplate object does not change any of his state information to process HTTP it can be considered thread safe so the same instance can be shared among multiple processes.

If multiple generation's/verification's processes run at the same time on the same SpringBoot instance different instances of Mac class are used, Mac objects are stateful so they can't be used by multiple threads.

## Logging

The library uses ***SLF4J*** as logging facade system.

Visit official documentation for more details:

http://www.slf4j.org/docs.html

If you want to enable the library's logs you have to configure the logging level DEBUG for the package ***com.ffsec***

This is an example with Log4j:

```
log4j.logger.com.ffsec=DEBUG
```
