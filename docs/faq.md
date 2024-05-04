# Frequent questions


## CSRF NOT WORKING

Yes, it will not work because CSRF uses sessions to protect its forms and/or other sections. Without sessions, it is difficult for an attacker to achieve a CSRF attack.
To date there is only one standard proposal to solve the problem of third-party cookies, which is why the package does not support creating sessions and instead are instances.
To use the package you will need to disable csrf validation on all routes that directly affect the LTI.