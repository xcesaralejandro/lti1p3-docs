# Link with an LMS

To link an LMS with an LTI you must make registrations in the LMS and also in your tool. The LMS configuration will vary depending on it and you can review the following documentation: [CANVAS](https://canvas.instructure.com/doc/api/file.lti_dev_key_config.html) and [MOODLE](https://docs.moodle.org/404/en/Publish_as_LTI_tool).

## Login in tool panel
To register an LMS you must log in to the tool's administration panel. The access credentials are those that you have configured in your Laravel `.env` file. The keys are mentioned in the **Required settings** section.

`https://[YOUR_APP_LARAVEL_DOMAIN]/lti1p3/login`

## Connection points
When configuring the tool in your LMS you will be asked for a URL as a connection point, for all URLs (except JWKS) you must use the following URL replacing your domain.

`https://[YOUR_APP_LARAVEL_DOMAIN]/lti1p3/connect`

You will also be prompted for a connection point to the tool's public key, this may appear as JWKS. For this section you must use the following URL

`https://[YOUR_APP_LARAVEL_DOMAIN]/api/lti1p3/jwks`

**IMPORTANT** 

The connection point to obtain the JWKS must be public, that is, there must be access from your LMS to your connection point. If you are working locally with a domain that is not public to the Internet, the LMS will not be able to obtain the keys.

Some LMSs will allow you to enter the manual key instead of a connection URL. You can enter to the URL of your JWKS and from there obtain the key in JSON format to manually add it to the LMS. If you add the key manually instead of using an endpoint, the LMS will have the key and can work locally without having to publicly expose a JWKS URL.

Generating a public key requires that you add a private key to the package configuration file located in `config/lti1p3.php`.