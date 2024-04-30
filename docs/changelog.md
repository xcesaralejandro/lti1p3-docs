# Changelog

### Version 6.0.2

**1.- General** 

- remember token returns from the dead

***

### Version 6.0.1

**1.- General** 

- Update dependencies and their impact to migrate to PHP-JWT >6.0 and fix the security gap
- Update outdated model imports without the "Lti" prefix to load Models from the "App" namespace instead of the package.
- Change the signature of the methods in the LtiRolesManager trait to receive LtiContext instead of Context. Same for LtiUserRole.
- Remove unnecessary columns from the model and lti user migrations (password, remembertoken).

***

### Version 6.0.0

**1.- General** 

- Unused use statements are removed
- The seeders that created an administrator user are removed, now the package does not use seeders.

**2.- Migrations** 

- Add prefix in migrations for tables and attributes with lti1p3_ to avoid name collisions
- All migrations use anonymous classes
- The creation source columns are eliminated (LTI - MANUAL) now everything stored is purely LTI. Previously the package took over the users table and that column made the difference in the creation source, now lti users are stored in their own users table with the prefix lti1p3_.

**3.- Models** 

- Models are prefixed with "Lti" to avoid name collisions
- The "isToolAdmin" method is eliminated in the user model since access to the dashboard is now with global credentials.

**4.- Instances** 

- Soft delete is added
- Since find and findOrFail can be perfectly used to search for an instance with the identifier, "RecoveryFromId()" is eliminated and, failing that, the relationships loaded by said method (platform, context, resource_link, user) are loaded by default in the model declaration

**5.- Non-lti authentication**

In previous versions, the package took over the concept of users and managed both those created through LTI and those that could be created manually in any other context of the application. Subsequently, resources such as the Auth facade were used to implement a login that allowed access to the LMS registration panel. This approach was incorrect as it restricted the functionality of mixed users (LTI and non-LTI). Additionally, building a custom login required overriding some functionality that the package was hijacking by managing users in a standard way.

Currently, we are looking to disassociate LTI users from the actions you want to perform in your application. For this reason, administration of the LTI panel will be carried out using a global user and password defined in its environment file and the lti users will be stored in a different table.

**6.- Routes**

- Routes and their names are prefixed with lti1p3
- The redirection of the login route to LTI authentication is eliminated
- Dashboard access is now: {YOUR_DOMAIN}/lti1p3/login

**6.- Middleware**

- The old "lti_instance_recovery" is renamed to "inject_lti_instance"
- A new middleware "lti1p3_session" is added to check if the user is logged in as the LTI administrator

**7.- Validation of initial content in the LTI protocol**

The user parameters: name, given_name, family_name and lis can be null, since the LTI can be configured to pass user data anonymously and that data is not sent, in previous versions of the package they are being required, which is why , when that configuration was given the LTI did not start throwing an exception.

**8.- Vistas**
- Migration from Material design bootstrap to Bootstrap 5.3.x
- Redesign of the administration panel for platforms
- Ability to edit a platform without having to delete it to recreate it

***
### Version 5.1.0

**1.- Now the user is authenticated with the middleware lti_instance_recovery** 

Having the user data in the instance is important, but it wasn't enough to work with policies and gates. Now, the middleware will also take care of authenticating the user automatically.

***
### Version 5.0.1

**1.- Fix input name when renderize the instance id with the blade directive** 

The blade directive renders a hidden input for you which is then retrieved by the middleware using a specific identifier. After the previous update, we forgot to update the key name for this directive.

***
### Version 5.0.0


**1.- Standardize the http code of responses when use lti_instance_recovery middleware** 

In previous versions, passing an incorrect instance could generate 404 responses instead of 401 (unauthorized) among other security problems.
Now if the token is not valid you will always get a 401 which is easier for the client to control.

**2.- Standardize instance key for lti_instance_recovery middleware** 

In older versions you could pass an instance identifier in the headers or as part of the request. These identifiers were similar but not the same, now for both cases the key is the same. 
````
// key name before (HEADERS):
lti1p3-instance-id

// key name before (REQUEST)
lti1p3_instance_id

// key name now, for both cases (HEADERS - REQUEST)
lti1p3-instance-id
````

***
### Version 4.1.0


**1.- Add leeway for check if token content is expired** 

Sometimes a lag of seconds could lead to a token expiration just because a request took longer than usual, that doesn't mean the token is actually expired. A maneuver time has been included by default to check if the token is expired or not (240s).
***

### Version 4.0.1


**1.- Fix publishing path for views and translations** 

The published views and translations do not overwrite the package files, this was because the call and publish paths were different.

***

### Version 4.0.0


**1.- Instances have expiration time** 

You may want to use the instances that contain all the necessary context and user information as an authentication token, considering that only one UUID is valid for the client and you have all the information in your backend. In practice, it is not necessary to use the instances and a different authentication system, because the instance will only be created when the LTI launch is correct, in other words, the LMS is in charge of verifying that the user is real.

Now, in the package configuration there is a new key that allows to define null or a number of seconds to determine the durability of the instance, after that, the application will return a 401 to all the routes that it protected with the ````lti_instance_recovery```` middleware.

***
### Version 3.1.1

**1.- Prefixed resources** 

You may want to have your own ````style.css```` file and forget about package styles. Now our ````style.css```` is called ````lti1p3_style.css```` to avoid potential collisions.
Other unused css files were removed.

***
### Version 3.1.0

**1.- New functions for get custom vars passed from LMS** 

In the content class, in charge of accessing the initial JWT message in a more friendly way, you will now have methods that will allow you to access and check custom variables established in the LTI configuration within the LMS.

````
    public function getAllCustomVars() : ?object 
    public function getCustomVar(string $name) : mixed
    public function hasCustomVar(string $name) : bool
    public function getCustomVarOrFail(string $name) : mixed
````

The method names are self-descriptive, just keep in mind that the failure for ````getCustomVarOrFail```` is an Exception.

**2.- Remove optional functions** 

These functions don't really make much sense, they returned ````null```` when a variable didn't exist and you had to check that later. We focus on what is important and leave the control of the optionality to you with the syntax of php 8.

````
    // Removed functions in content class.
    public function optionalResourceLinkAttribute(string $attribute) : mixed
    public function optionalPlatformAttribute(string $attribute) : mixed
````

Now you can do something like this:

````
// reference
$content->getPlatform()->some_optional_var ?? whatever;
// Example of actual usage
$validation_context = $content->getPlatform()->validation_context ?? null;
````


***
### Version 3.0.2

**1.- Remove unused importation** 

Removed an unused import in the launch controller. The import could be confusing and imported the Instance model directly from the package instead of the published model.


***
### Version 3.0.1

**1.- Fix roles** 

Since the current user is no longer authenticated and the way roles are stored was reformulated, the roles trait was corrupted. Now methods that query for roles require a context instance to query for roles in a particular context.

**2.- Fix models importation** 

Some model imports kept referencing the original models in the package and not the ones published within your application. If you wanted to add methods or new things in general they would never work because in reality the published model was not the one that was being called.
Another issue identified was that passing an instance of a model could fail due to the model having a different namespace.


***
### Version 3.0.0

**1.- Deep linking support.** 

Support for deep link launches is added. This generates changes in the LTI launch methods (See point 2). A new class is implemented to generate responses of type _Deep Linking_ and can be found at: ````xcesaralejandro\lti1p3\Classes\DeepLinkingResponse````.

**2.- Goodbye to the launch method onLaunch()** 

Previously you only had to worry about starting your application in the ````onLaunch()```` method, there the initial message was already signed and the message type was ignored and treated as a resource hook due to lack of support. for _Deep Linking_. The method is now renamed based on the initial message type based on the standard. ````onLaunch()```` is now called ````onResourceLinkRequest()```` and ````onDeepLinkingRequest()```` is added. If your application doesn't use _Deep Linking_, handling the launch in ````onResourceLinkRequest()```` should suffice. 

**3.- Hello launch ids** 

Since sessions were easily lost or overwritten when launching the same LTI in multiple browser tabs, maintaining LTI launch data was a problem. Now items related to the LTI release are stored in the database under an identifier that will allow the information to be retrieved later, the identifier must be passed constantly between URL changes.

**4.- Aids for handling identifiers**

Constantly passing the identifier between URLs and then retrieving the data is somewhat tedious, so a middleware called ````lti_instance_recovery```` appears that will verify the existence of a parameter called ````lti1p3_instance_id```` inside the ````Request````, if it exists, will retrieve the instance and include it within the same ````Request````. You can also use ````Instance::RecoveryFromId($ID)```` to retrieve data from an instance manually.

Passing the instance identifier to the view shouldn't be a problem, but if we have a lot of forms creating _inputs hidden_ can be inelegant. Failing that, use the ````@addinstance($instance_id)```` directive which will add an _input hidden_ for you.

***
### Version 2.0.0

**1.- Corrects the implementation of "deployments".** 

Previously, the deployment was linked to the platform registry, this limited the lti to be installed only once within the LMS (what a mistake...).
Now the deployment record was separated into its own table -as specified by the standard- and with this it is possible to install the LTI globally, in specific courses or as you wish.
The admin panel has been updated to support this change.

**2.- User roles are now stored locally**

Role validations are still performed at runtime, but now the roles are being stored in a table for record keeping (These can become out of date if a role is updated in the LMS and the user does not log back into the LTI) . It is possible that roles from higher contexts are repeated, since they transcend to a lower level context and currently all the roles passed by the LMS are saved, associating them with their launch context.

Roles were split into a new table and removed as a column from users.

**3.- Goodbye deleted records**

Soft deletes have been implemented for all models.

**4.- Nod to LTI Advantage**

Added a new field for platform registration: ````lti_advantage_token_url```` this allows defining an endpoint to obtain lti advantage tokens. Even though it's not implemented yet, it puts pressure on me and reminds me to keep developing. 