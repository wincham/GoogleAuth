[![Travis build status](https://travis-ci.org/ari/GoogleAuth.svg?branch=master)](https://travis-ci.org/ari/GoogleAuth)
[![License](https://img.shields.io/badge/license-BSD-blue.svg?style=flat)](https://github.com/wstrange/GoogleAuth/blob/master/LICENSE)

README
======

GoogleAuth is a Java server library that implements the _Time-based One-time
Password_ (TOTP) algorithm specified in [RFC 6238][RFC6238].

This implementation borrows from [Google Authenticator][gauth], whose C code has
served as a reference, and was created upon code published in
[this blog post][tgb] by Enrico M. Crisostomo.


Whom Is This Library For
------------------------

Any developer who wants to add TOTP multi-factor authentication to a Java
application and needs the server-side code to create TOTP shared secrets
and verify TOTP passwords.

Users may use TOTP-compliant token devices (such as those you get from your bank),
or a software-based token application (such as Google Authenticator).

Installing
----------

Add a dependency to your build environment.

If you are using Maven:

    <dependency>
      <groupId>com.warrenstrange</groupId>
      <artifactId>googleauth</artifactId>
      <version>0.5.0</version>
    </dependency>

If you are using Gradle:

     compile 'com.warrenstrange:googleauth:0.5.0'

The required libraries will be automatically pulled into your project:

  * Apache Commons Codec.
  * Apache HTTP client.

The minimum Java version required to build and use this library is Java 7.

Client Applications
-------------------

Both the Google Authenticator client applications (available for iOS, Android
and BlackBerry) and its PAM module can be used to generate codes to be validated
by this library.

However, this library can also be used to build custom client applications if
Google Authenticator is not available on your platform or if it cannot be used.

Library Documentation
---------------------

This library includes full JavaDoc documentation and a JUnit test suite that can
be used as example code for most of the library purposes.

Texinfo documentation sources are also included and a PDF manual can be
generated by an Autotools-generated `Makefile`:

  * To bootstrap the Autotools, the included `autogen.sh` script can be used.
  
  * Configure and build the documentation:

        $ ./configure
        $ make pdf

Since typical users will not have a TeX distribution installed in their
computers, the PDF manuals for every version of GoogleAuth will be hosted at
[this address][pdfdoc].

[pdfdoc]: https://drive.google.com/folderview?id=0BxZtP9CHH-Q6TzRSaWtkQ0pEYk0&usp=sharing

Usage
-----

The following code creates a new set of credentials for a user. No user name is
provided to the API and it's responsibility of the caller to save them for later
use during the authorisation phase.

    GoogleAuthenticator gAuth = new GoogleAuthenticator();
    final GoogleAuthenticatorKey key = gAuth.createCredentials();

The user should be now given the key.getKey() value to load into their token device. A convenience method is provided to easily convert that into a QRcode. That key also needs to be stored within the application data storage.


When a user wishes to log in, they will provide a token generated from the secret key. This token is typically a 6 digit integer and changes every 30 seconds by default.

The following code checks the validity of the specified `token` against the
provided Base32-encoded `secretKey`:

    GoogleAuthenticator gAuth = new GoogleAuthenticator();
    boolean isCodeValid = gAuth.authorize(secretKey, token);

It is essential that the system clock is accurate to within a few seconds for this system to work properly. Review NTP server options for the operating system being used.


Scratch codes
-------------
By default 5 scratch codes are generated which are a bypass mechanism for users who have lost their token generating device. It is up to the developer to store those scratch codes and remove them as they are used.


Storing User Credentials
------------------------

The library can assist with fetching and storing user credentials and a hook is
provided to users who want to integrate this functionality. The *ICredentialRepository* interface defines the contract between a credential repository and this library.

The library looks for instances of this interface using the
[Java ServiceLoader API][serviceLoader] (introduced in Java 6), that is,
scanning the `META-INF/services` package looking for a file named
`com.warrenstrange.googleauth.ICredentialRepository` and, if found, loading the
provider classes listed therein.

Two methods needs to be implemented in the *ICredentialRepository* interface.

  * `String getSecretKey(String userName)`.
  * `void saveUserCredentials(String userName, ...)`.

The credentials repository establishes the relationship between a user _name_
and its credentials.  This way, API methods receiving only a user name instead
of credentials can be used. Instead of gAuth.createCredentials() you can use gAuth.createCredentials(username). Instead of 

The following code creates a new set of credentials for the user `Bob` and
stores them on the configured `ICredentialRepository` instance:

    GoogleAuthenticator gAuth = new GoogleAuthenticator();
    final GoogleAuthenticatorKey key = gAuth.createCredentials("Bob");


The following code checks the validity of the specified `code` against the
secret key of the user `Bob` returned by the configured
`ICredentialRepository` instance:

    GoogleAuthenticator gAuth = new GoogleAuthenticator();
    boolean isCodeValid = ga.authorizeUser("Bob", code);


If an attempt is made to use such methods when no credential repository is
configured, a meaningful error is emitted:

    java.lang.UnsupportedOperationException: An instance of the
    com.warrenstrange.googleauth.ICredentialRepository service must be
    configured in order to use this feature.

Bug Reports
-----------

Please open a ticket on [github][githubIssues].

[RFC6238]: https://tools.ietf.org/html/rfc6238
[gauth]: https://code.google.com/p/google-authenticator/
[tgb]: http://thegreyblog.blogspot.com/2011/12/google-authenticator-using-it-in-your.html?q=google+authenticator
[serviceLoader]: http://docs.oracle.com/javase/6/docs/api/java/util/ServiceLoader.html
[SecureRandom]: http://docs.oracle.com/javase/8/docs/api/java/security/SecureRandom.html
[sr-algorithms]: http://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html#SecureRandom
[githubIssues]: https://github.com/wstrange/GoogleAuth/issues
