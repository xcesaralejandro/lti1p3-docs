# About LTI

## What is LTI 1.3?

It is a standard that empowers learning management platforms (LMS) to incorporate new functionalities. Managed by 1EdTech (Previously called IMS Global), this standard facilitates the transfer of data and content in a uniform and compatible manner between different educational systems. This interoperability is essential to ensure a consistent and seamless user experience, while encouraging innovation and expansion of the capabilities of digital educational platforms.

LTI 1.3 is the core of the standard, there are also different services that allow extending the functionality, these are included within a package known as LTI Advantage.

## LTI Advantage

This service is made up of different complements that seek to strengthen the LTI core, these are:

**Names and Role Provisioning Services (NRPS)**

Allows you to obtain the members of a course and their respective roles. If users are grouped, they will be returned in the respective groups.

**Deep Linking**

Allows you to add existing resources to the tool or build a resource from a previously defined configuration.

**Assignment and Grades Services (AGS)**

Allows you to create gradeable tasks and assign grades. The modification of notes is restricted to the scope of action of the LTI.

**LTI Platform Storage**

Allows a tool to store temporary values within a window initialized by a platform. This definition seeks to solve the problem of third-party cookies.


## Going deeper into the standards

| STANDARD | URL |
|----------|----------|
| LTI 1.3 | [Go to standard](https://www.imsglobal.org/spec/lti/v1p3) |
| Deep Linking | [Go to standard](https://www.imsglobal.org/spec/lti-dl/v2p0) |
| NRSP | [Go to standard](https://www.imsglobal.org/spec/lti-nrps/v2p0) |
| AGS | [Go to standard](https://www.imsglobal.org/spec/lti-ags/v2p0) |
| LTI Platform Storage | [Go to standard](https://www.imsglobal.org/spec/lti-pm-s/v0p1) |
| LTI Advantage | [Go to standard](https://www.imsglobal.org/lti-advantage-overview) |


**Currently this package only has support for the LTI 1.3 core and LTI Deep Linking**. In the development branch there is a proof of concept to consume other LTI Advantage services, that works in Moodle but we have encountered difficulties for canvas, once resolved we will support it in a more standardized way.
