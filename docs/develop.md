# Time to develop


### Entry point

The application will publish models and controller, you can start developing from there.

After registering a platform (LMS), the package will be in charge of synchronizing and updating all the data coming from the LMS or creating it if it does not exist. With this data we refer to the information of the platform, course, resource, user, etc.

Once the launch has concluded, the LtiController is called in its different methods depending on the final state of the flow.

````
<?php

namespace App\Http\Controllers;

use xcesaralejandro\lti1p3\Http\Controllers\Lti1p3Controller;

class LtiController extends Lti1p3Controller {

    /*

        Important!
        Consider that an LTI can be added on multiple sides, 
        sometimes your LTI can receive LtiResourceLinkRequest and LtiDeepLinkingRequest triggers

    */

    public function onResourceLinkRequest(string $instance_id) : mixed {
        return parent::onResourceLinkRequest($instance_id);
        // Do something, here it is not necessary to call the parent function,
        // it is only to maintain the example functionality

        // This method is called when the lti launch is of type LtiResourceLinkRequest
        // You can read about it here: http://www.imsglobal.org/spec/lti/v1p3/#resource-link-launch-request-message

        // In human words, it is the launch of the LTI after doing the validations behind the scenes and the synchronization 
        // of the data that arrives from the LMS with the local platform (The one that you must now start developing). 
        // Sometimes this launch can be skipped by a custom redirect if you defined it in the LMS or it is a LtiDeepLinkingRequest.
    }

    public function onDeepLinkingRequest(string $instance_id) : mixed {
        return parent::onDeepLinkingRequest($instance_id);
        // Do something, here it is not necessary to call the parent function,
        // it is only to maintain the example functionality

        // This method is called when the lti launch is of type LtiDeepLinkingRequest
        // You can read about it here: https://www.imsglobal.org/spec/lti-dl/v2p0#overview

        // In human words, it is the launch of the LTI when it comes to DeepLinking. This, depending on the location where you add your LTI, 
        // allows you to generate custom resources, return tasks, among other things.
        // I recommend reading the specification because to date I haven't tested everything it allows (UPS!).

        // In such a launch you must reply back to the LMS with a DeepLinking message to end the cycle.
        // You can browse the original model to review how the example works.
    }

    public function onError(mixed $exception = null) : mixed {
        return parent::onError($exception);
        // Do something, here it is not necessary to call the parent function,
        // it is only to maintain the example functionality

        // The onError method will be thrown when an invalid connection is attempted, 
        // something goes wrong in the launch process (LMS-LTI). If it is the latter case, 
        // it is most likely due to some problem with the configuration.
        
        // If you're sure you've set everything up correctly and you're still getting errors, 
        // open a github bug, I'll be happy to cry with you for not understanding what's wrong.
    }
}

````

At launch you will receive an instance that will allow you to have all the information regarding the launch. The methods of this controller are also a good point to consult extra information such as custom variables. Once you have reached this point and have made sure that you have all the necessary values, you can redirect to another controller or even another domain, passing the data in the way that best suits you, from there you can build an application as you like. knows how to do it.


## Understanding the instance_id

**What the heck is the instance id?**

LTIs are loaded in iframes, which means that all cookies are considered third-party and, based on new browser policies, are blocked. This problem manifests itself especially in the context of sessions, since the session identifier, stored in a cookie on the client side by the server, would not persist.

Additionally, the package is designed to address various needs that may arise with an LTI, such as a resource in a virtual classroom. If a course has two or more LTI resources, and they are opened independently one after another, the session would be overwritten. A possible solution would be to manage a session and store data according to the open tab, although this modifies the traditional concept of a session and can lead to errors at runtime.

Although we have found other details in the process, these two elements were decisive in choosing to create an instance. After various trials and errors, we sought a friendly implementation. The instance includes everything necessary to access what is provided by the LTI, while offering elements to use it as an obfuscated security token (UUID).

Instances are generated upon launch; That is, if a person opens the LTI in two different tabs, he will have two different instances. If the person reloads the page while in a browser tab, a new instance will be created.

This instance-as-identifier approach is very useful in the context of backend for frontend (BFF), webservices, or other concepts that require an authentication token. To use Laravel as a server-side renderer (SSR) and still use Laravel as usual (possibly with Blade), you can add and retrieve the instance identifier in your forms using the controller.

Also note the existence of middlewares and the Blade directive, which will make it easier to pass and obtain the instance ID when working in SSR mode.


Retrieve an instance from the handle
````
use App\Models\LtiInstance;
$instance = LtiInstance::find($instance_id);
````


If you are working with SSR you will have to constantly share your instance identifier between the different forms, you can use the following blade helper to add your instance, this will create an entry with a name defined by the package and then recover the instances with a middleware.

````
@addinstance($instance_id)
````

If you add the instance middleware, it will attempt to retrieve a key called `lti1p3-instance-id` from the request headers or parameters and initialize the model to include it in your request.

````
inject_lti_instance
````

At this point, you can add the middleware globally and rely on the blade directive to render the identifier of your instance.


## Current user roles

Before starting with the functions that the package provides, it is necessary to understand the roles that exist in the vocabulary defined in the LTI 1.3 standard. 

[See role vocabularies](http://www.imsglobal.org/spec/lti/v1p3/#role-vocabularies)


If we look at the role tables, these are made up of four scopes: system, institution, membership and person (Although the latter are deprecated, they can still exist). Below, in the point "A.2.3.1 Context sub-roles" a grouping of roles is shown without deepening the scope, with a logic of main role and sub-roles, from there the package is based to make some functions available that will allow you to check the role of a user.

Eventually the roles of a user can vary depending on the context, while others are general. The package is transparent and on each launch it will store all the roles passed by the LMS and associate them to the context where the LTI is being launched, even if it is a higher context role.

Note that there may be a mismatch between the current roles in the LMS and those stored in the LTI. If a user does not launch the LTI, it has no way of knowing the user's current roles. Therefore, if a user does not re-enter the LTI anymore and their roles change, there will be a lag for the roles that are stored.

Roles will always be updated for the user in the LTI when it is launched and you can use LTI Advantage Names and Roles Service (NRPS) for update the roles without depend of user access.

Behind the scenes, user roles are managed by a different model than user, eventually you will be able to bring in all the roles of a user or work with eloquent as usual.
If you are working in a particular context, you will need the roles of that context, so these thoughtful methods for managing LTI roles will help make your life easier.

Once you query the roles of a context, the model will keep them inside an array in the respective class instance to avoid multiple queries, since the roles should change (update) only on LTI launch.

With this in mind, we describe methods that will help you manipulate roles.

The ````$user```` variable is an instance of the ````LtiUser```` model. The ````$context```` variable is a reference to the ````LtiContext```` model.

````
$user->isAdmin($context)
$user->isContentDeveloper($context)
$user->isInstructor($context)
$user->isLearner($context)
$user->isManager($context)
$user->isMentor($context)
$user->isOfficer($context)
$user->isMember($context)
$user->isTestUser($context)
````

By default, the above functions will search for sub-roles in all scopes (system, institution, membership and person), the sub-roles for each "Main Role" are defined as an array within the package configuration file, for which, you can modify them from there in case you need it.
The user class has a trait that takes care of the roles and has some static properties that will serve to pass as a scope filter to the previous functions.

````
public static $SYSTEM_ROLES = 'http://purl.imsglobal.org/vocab/lis/v2/system/person#';
public static $INSTITUTIONAL_ROLES = 'http://purl.imsglobal.org/vocab/lis/v2/institution/person#';
public static $CONTEX_ROLES = 'http://purl.imsglobal.org/vocab/lis/v2/membership#';
public static $PERSON_ROLES = 'http://purl.imsglobal.org/vocab/lti/system/person#';
````

Then, you can query for a main role in a particular scope as follows:

````
use App\Models\LtiUser;
$user->isAdmin($context, LtiUser::$CONTEX_ROLES)
user->isContentDeveloper($context, LtiUser::$CONTEX_ROLES)
$user->isInstructor($context, LtiUser::$CONTEX_ROLES)
$user->isLearner($context, LtiUser::$CONTEX_ROLES)
$user->isManager($context, LtiUser::$CONTEX_ROLES)
$user->isMentor($context, LtiUser::$CONTEX_ROLES)
$user->isOfficer($context, LtiUser::$CONTEX_ROLES)
$user->isMember($context, LtiUser::$CONTEX_ROLES)
$user->isTestUser($context, LtiUser::$CONTEX_ROLES)
````

Now we will list some functions that may be useful to you.

**"Friendly role": It is the role without the specification prefix**


Returns the roles in a friendly and unique way, this means that if the role is repeated in different scopes, it will only be shown once.

````
$user->getAllRoles($context)
````


Returns the system roles in a friendly way

````
$user->getSystemRoles($context)
````


Returns institutional roles in a friendly way

````
$user->getInstitutionRoles($context)
````


Returns context roles in a friendly way

````
$user->getContextRoles($context)
````


Return person roles in a friendly way

````
$user->getPersonRoles($context)
````


It receives an array of roles and a context (understand context as the static variables that contain the scope).
Returns a boolean in case of having any of the roles, if a context is not passed it will be searched in all contexts.

````
$user->hasSomeRolesOf(Context $context, array $role_list, ?string $role_context=null)
````


**Example**

It will look for the roles in all scopes and returns true if it finds any

````
$user->hasSomeRolesOf($context, ['Administrator', 'Instructor'])
````


It will look for the roles in the context scope and return true if it finds any

````
$user->hasSomeRolesOf($context, ['Administrator', 'Instructor'], LtiUser::$CONTEX_ROLES)
````


It receives a role name and a context (understand context as the static variables that contain the scope).
Returns a bolean in case of having the role, if a context is not passed it will be searched in all contexts.

````
$user->hasRole(Context $context, string $role_name , ?string $role_context = null)
````


**Example**

Will look for the role in all scopes

````
$user->hasRole($context, 'Administrator')
````


Will look for the role in the person scope

````
$user->hasRole($context, 'Administrator', LtiUser::$CONTEX_ROLES)
````

## Get data from initial message

The LTI launch data is contained in a JWT sent as an initial message. This message can be retrieved from the instance managed by the package.

The package uses some classes that you can call to work with the initial message and retrieve some more "raw" information.

````
use App\Models\LtiInstance;
use xcesaralejandro\lti1p3\Classes\Content;
use xcesaralejandro\lti1p3\Classes\Message;

public function onResourceLinkRequest(string $instance_id) : mixed {
    $instance = LtiInstance::findOrFail($instance_id);
    $content = Message::decodeJWT($instance->platform, $instance->initial_message);
    // Get custom vars
    $some_var_1 = $content->getCustomVar('VAR_DEFINED_IN_THE_LMS');
    $some_var_2 = $content->getCustomVarOrFail('VAR_DEFINED_IN_THE_LMS');
    // Recovery initial message
    $raw_message = $content->getRawJwt();
    // Redirect to your app logic
    return (new LaunchController)->appStart($instance_id);
}
````

As seen in the previous example, we recover the instance and from the Message class we decode the initial message, then we access some methods that will allow us to obtain personalized variables in a better way and another method that allows us to obtain the initial message with all the information already decoded .
Finally we redirect to another point in our application.

To learn more methods you can review the public methods in the `Content` class. Many times you will only need custom variables, since the rest of the values are taken care of by the package in the respective published models.


## Middlewares

| Name  | Description  |
|---|---|
| lti1p3_session  | Check if the user is logged in as the LTI administrator  |
| inject_lti_instance  | Adds an `LtiInstance` instance to the request when a key `lti1p3-instance-id` <br/> exists in the headers or parameters of the request.  |