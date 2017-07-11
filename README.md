# Kerberos.NET
A Managed Code validator for Kerberos tickets

# Getting Started
There are two ways you can go about using this library. The first is to download the code and build it locally. The second, better, option is to just use nuget.

```powershell
PM> Install-Package Kerberos.NET
```

This will get you the main Kerberos parser and RC4 support, which (sadly) is all people really need in most environments because RC4 is so pervasive still. However, if you want to support newer, better, slightly more secure, algorithms you need to install the AES package as well.

```powershell
PM> Install-Package Kerberos.NET-AES
```

The AES package is separated from the main package because it has dependencies on BouncyCastle.

## Using the Library

The library has a number of classes available to you to get deep in to the Kerberos-isms, but I suspect most people only care about validation so all you really need to do is create a validator and Validate the incoming token:

```C#
var validator = new SimpleKerberosValidator(new KerberosKey("P@ssw0rd!", host: "server01"));

var identity = validator.Validate("YIIHCAYGKwYBBQUCoIIG...");

Assert.IsNotNull(identity);

var name = identity.Name;

Assert.IsFalse(string.IsNullOrWhitespace(name));
```

Including AES support is easy. Just register the decryptors during app startup.

```C#
AESKerberosConfiguration.Register();
```

This registration is also a good example for how you can add your own support for other algorithms like DES (don't know why you would, but...) where you associate an Encryption type to a Func<> that instantiates new decryptors. There's also nothing stopping you from DI'ing this process if you like.

```C#
KerberosRequest.RegisterDecryptor(
   EncryptionType.DES_CBC_MD5,
   (token, key) => new DESMD5DecryptedData(token, key)
);
```

Note that the existing replay detection used internally is just a HashSet<string> detecting whether the incoming token has been seen before. There's a TODO item to make this more useful as this doesn't remove the strings after a period of time. If you'd like touse your own replay detection just implement the `ITicketCacheValidator` interface and pass it in the `SimpleKerberosValidator` constructor.

# Samples!
There are samples!

 - [KerbCrypto](https://github.com/SteveSyfuhs/Kerberos.NET/tree/master/KerbCrypto) Runs through the 6 supported token formats.
    - rc4-kerberos-data
    - rc4-spnego-data
    - aes128-kerberos-data
    - aes128-spnego-data
    - aes256-kerberos-data
    - aes256-spnego-data
 - [KerbTester](https://github.com/SteveSyfuhs/Kerberos.NET/tree/master/KerbTester) A command line tool used to test real tickets and dump the parsed results.
 - [KerberosMiddlewareEndToEndSample](https://github.com/SteveSyfuhs/Kerberos.NET/tree/master/KerberosMiddlewareEndToEndSample) An end-to-end sample that shows how the server prompts for negotiation and the emulated browser's response.
 - [KerberosMiddlewareSample](https://github.com/SteveSyfuhs/Kerberos.NET/tree/master/KerberosMiddlewareEndToEndSample) A simple pass/fail middleware sample that decodes a ticket if present, but otherwise never prompts to negotiate.
 - [KerberosWebSample](https://github.com/SteveSyfuhs/Kerberos.NET/tree/master/KerberosWebSample) A sample web project intended to be hosted in IIS that prompts to negotiate and validates any incoming tickets from the browser.

# The TODO List
Just a list of things that should be done, but aren't yet.

 - Replay detection is weak. The default `ITicketCacheValidator` needs to be a proper LMU cache so older entries are automatically cleaned up. The detection should also probably happen after a partial decoding so we can use the ticket's own expiry to remove itself from the cache. The validator should be simple enough that it can be backed by shared storage for clustered environments.
 - Validation and transformation isn't extensible. It just dumps a ClaimsIdentity with a fairly arbitrary list of claims from the ticket. You should be able to easily get whatever information you want out of the token. Validation also shouldn't be disabled so easily (it's currently just a bool flag on the validator class).
 - The validation process should be vetted by someone other than the developer.

# License
This project has an MIT License, but both the RC4 and MD4 implementations are externally sourced and have their own license.

The crypto data and key test files were sourced from https://github.com/drankye/haox/ originally under Apache 2.0 license https://github.com/drankye/haox/blob/master/LICENSE