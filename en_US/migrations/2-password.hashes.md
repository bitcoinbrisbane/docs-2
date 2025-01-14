# Password hashes

!!! attention

	 Check out the lightweight on-premises email archiving software developed by iRedMail team: [Spider Email Archiver](https://spiderd.io/).

[TOC]

## Password hashes supported by iRedMail

iRedMail configures Postfix to use Dovecot as SASL authenticate server, so all
password schemes supported by Dovecot can be used in Postfix. Please refer to
Dovecot wiki page
[`Password Schemes`](http://wiki2.dovecot.org/Authentication/PasswordSchemes)
for more details.

Below password schemes are supported in iRedAdmin-Pro (which means you can add new mail user with either one):

1. SSHA512. e.g. `{SSHA512}FxgXDhBVYmTqoboW+ibyyzPv/wGG7y4VJtuHWrx+wfqrs/lIH2Qxn2eA0jygXtBhMvRi7GNFmL++6aAZ0kXpcy1fxag=`
1. BCRYPT. e.g. `{CRYPT}$2a$05$TKnXV39M3uJ4o.AbY1HbjeAval9bunHbxd0.6Qn782yKoBjTEBXTe`
1. SSHA. e.g. `{SSHA}OuCrqL2yWwQIu8a9uvyOQ5V/ZKfL7LJD`
1. MD5 (salted). For example:

    * with a prefix: `{CRYPT}$1$GfHYI7OE$vlXqMZSyJOSPXAmbXHq250`
    * without a prefix: `$1$GfHYI7OE$vlXqMZSyJOSPXAmbXHq250`

    __Important note__: SOGo groupware doesn't support MD5 without a prefix, so
    if you're going to migrate MD5 password hash from old mail server, please
    prepend `{CRYPT}` prefix in password hash.

1. PLAIN-MD5 (without a salt). e.g. `0d2bf3c712402f428d48fed691850bfc`
1. Plain text. e.g. `123456`

__WARNING__: MD5, PLAIN-MD5 and plain password are weak, please don't use them.

__NOTES__:

* `BCRYPT` is only available on BSD systems, because `libc` shipped in Linux
  doesn't support bcrypt.

## Default password schemes used in iRedMail

* For MySQL and PostgreSQL backends:

    * in iRedMail-0.9.0 and later versions: `SSHA512`
    * in iRedMail-0.8.7 and earlier versions: `salted MD5`

* For LDAP backends:
    * in iRedMail-0.9.5 and later versions:
        * Debian 8, Ubuntu 16.04, FreeBSD: `SSHA512`
        * RHEL/CentOS 6/7, Ubuntu 14.04, OpenBSD: `SSHA`. OpenLDAP package
          shipped in these distributions don't support SHA-2 password
          verification by default.
    * in iRedMail-0.9.4 and earlier versions: `SSHA`.

    !!! note

        OpenLDAP's builtin password verification doesn't support SHA-2 password
        hash formats directly, so if you have third-party applications which need
        OpenLDAP's builtin password verification, you'd better use `SSHA` hash.

        If you don't have such concern, it's ok to store `SSHA512/BCRYPT`
        hash as mail user password, then set `ldap_bind = no` in
        `/etc/dovecot/dovecot.conf`. SMTP/IMAP/POP3 services work with it, but
        Apache basic auth doesn't.

## How to use different password hashes in iRedMail

### For MySQL and PostgreSQL backends

All mail users are stored in SQL table `vmail.mailbox`, user password is stored
in SQL column `mailbox.password`. For example (Note: you should replace `xx@xx`
with your real email address):

```
sql> USE vmail;

sql> UPDATE mailbox SET password='$1$GfHYI7OE$vlXqMZSyJOSPXAmbXHq250' WHERE username='xx@xx';
sql> UPDATE mailbox SET password='{SSHA}OuCrqL2yWwQIu8a9uvyOQ5V/ZKfL7LJD' WHERE username='xx@xx';
sql> UPDATE mailbox SET password='{SSHA512}FxgXDhBVYmTqoboW+ibyyzPv/wGG7y4VJtuHWrx+wfqrs/lIH2Qxn2eA0jygXtBhMvRi7GNFmL++6aAZ0kXpcy1fxag=' WHERE username='xx@xx';
```

* To store PLAIN-MD5, you have to prepend `{PLAIN-MD5}` in your password hash:

```
sql> USE vmail;
sql> UPDATE mailbox SET password='{PLAIN-MD5}0d2bf3c712402f428d48fed691850bfc' WHERE username='xx@xx';
```

* To store plain password, you have to prepend `{PLAIN}`:

```
sql> USE vmail;
sql> UPDATE mailbox SET password='{PLAIN}123456' WHERE username='xx@xx';
```

### For OpenLDAP backend

User password is stored in attribute `userPassword` of user object.

* To store plain password, SSHA, SSHA512 password hash, just store them in
original format. For example:

```
userPassword: 123456
userPassword: {SSHA}OuCrqL2yWwQIu8a9uvyOQ5V/ZKfL7LJD
userPassword: {SSHA512}FxgXDhBVYmTqoboW+ibyyzPv/wGG7y4VJtuHWrx+wfqrs...
```

* To store standard MD5 password (salted MD5 hash), please prepend `{CRYPT}`
(case insensitive) in your password hash. For example:
```userPassword: {CRYPT}$1$GfHYI7OE$vlXqMZSyJOSPXAmbXHq250```

__IMPORTANT NOTE__: If you want to input password hash with phpLDAPadmin,
please choose `clear` in the password hash list, then input password hash.

## See also

* [Reset user password](./reset.user.password.html)
