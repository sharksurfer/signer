# signer
SpringBoot library for automatic and secure http request signing.

This library provide you a way for signing your RestTemplate HTTP request accross microservices.
The library offers two annotations, @Sign and @Signed for client and server respectively.

@Sign
This annotation must be placed on the client method inside wich the http call is made with RestTemplate.
The library will add the necessary headers to the request that contain the signature, everything happens transparently for the user.

@Signed
This annotation must be placed on the called rest endpoint and tells to the library that this method is signed and the http request must be validated.
If the verification process completes succesfully, the http request will be handled by the server otherwise the client will receive a 401 UNAUTHORIZED response message.

Both the client and the server must have the same secret key configured inside them since the signature algorithm use a symmetric key.
On client side the secret key is used to generate the signature, on server side the same key is used for the singnature verification process.

The property to set is 'ffsec.signer.secret' and must contains a randomic generated string with any length (recommended 128/256/512 bit).

Is also possible for the user to define the hashing algorithm used for the signature generation, the default is SHA-256 but also MD5 and SHA-1 are supported.

The property for the hashing algorithm is 'ffsec.signer.algorithm' and the possible values are MD5, SHA-1 and SHA-256.

The library use a randomic seed for the signature generation, this seed is combined with the secret key and finally hashed, 
this process guarantee high security.
