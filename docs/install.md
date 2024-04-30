# Install

## Requeriments

**Php** <br /> >= 8.0

**Laravel** <br />>= 8.0


**Set up a domain name**<br /> You need configure a local domain to develop, since when using JWT, `localhost`, `127.0.0.1` or your current IP may not work correctly for signing. You can register a local domain name at: Linux: `/etc/hosts` and Windows: `C:\Windows\System32\drivers\etc\hosts`

**HTTPS Cert**<br />
You will need an https certificate to work with LTI, you can use a self-signed one. If you are using docker you can take a look at [traefik](https://doc.traefik.io/traefik/), otherwise you can look at [certbot](https://certbot.eff.org/) which will help you obtain a certificate more easily. If not, go for [Let's encrypt](https://letsencrypt.org/es/getting-started/)

## Installing LTI1P3

### Step 1- Add package
Use composer to install the dependency with the following command:

`composer require xcesaralejandro/lti1p3`

### Step 2 - Publish the provider
The package provider will publish controllers, models, migrations, views, and everything you need to get started. You can extend or modify the functionalities later from the published files.

`php artisan vendor:publish --provider="xcesaralejandro\lti1p3\Providers\Lti1p3ServiceProvider" --force`

### Step 3 - Run migrations
````php artisan migrate````

### Style problems
If you are using some reversible proxy it is possible that the package styles published by the provider will try to load with http instead of https, which produces an error. To fix the problem you can force the https scheme in your `app/Providers/AppServiceProvider.php`, adding ````\URL::forceScheme('https'); ```` in the `boot` method.


## Required settings

### Create admin panel credentials
Add the following keys to the Laravel `.env` file. These will allow you to log in to the administration panel and thereby register the LMS.

````
LTI1P3_ADMIN_USERNAME=example@lti1p3.cl
LTI1P3_ADMIN_PASSWORD=lti1p3_admin
````

### Update the package configuration file

After publishing our provider, we will have a file in `config/lti1p3.php`. There we must complete the configuration and we can manage the criteria to work with the roles.

The **required keys** within the file `lti1p3.php` are listed below, the rest should be the default unless you understand that you are changing.

| Key   |  Description | 
| :---  |  :---        |
| **VERIFY_HTTPS_CERTIFICATE** | If true, will not allow self-signed https certificates. |
| **KID** | An identifier invented by you, this will be used to <br> identify the public key in the Json Web Tokens (JWT) |
| **PRIVATE_KEY** | A private RSA key. This key is used to generate the Json Web Key <br> and encrypt the communication with the LMS. <br> You can generate a new RSA private key of 2048 bit here:<br> https://travistidwell.com/jsencrypt/demo/|