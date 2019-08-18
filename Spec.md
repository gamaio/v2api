# API Specification

The goal of this document is to lay out the goals and eventual featureset of this project. This document may be updated with newer versions as dictated by [Semantic Versioning](https://semver.org/); The changelog is recorded at the bottom of this document.  

The current API Specification Version is: 2.0.0  
The current Production API Version is: 1.0.0  

## Features

The features of the API are as follows:

1. Shorten a link (or return existing short link) - When provided a long link, it will check if it has been shortened before and return the short link if so, or generate a new short link. You won't be able to update the URL for 24 hours after creation
1. Resolve a link - Either a short link to long link, or long link to short link
1. Update a link - If the link is either owned by your User, or an Application has the correct permission, it will be able to change various information such as the expiry timer, protected status (password), or the long link itself (once per 24 hours)
1. Delete a link - If the link is either owned by your User, or an Application has the correct permission, it will be able to flag the link as deleted. It will no longer resolve to the long link, nor record new statistics. There will be a grace period of 24 hours to "Un-delete" the link (same as creating new links, you won't be able to update the URL for 24 hours). If the link is not "Un-deleted", the short id will be made available for other users, though your personal statistics will remain (unless deleted)
1. Link Statistics - Information about when a link was created, how many total clicks, how many unique (per IP and User Agent combination) clicks, when it expires, how many times it was deleted, when it was deleted, the city name of accessing IP. IP address information is not available to users.
1. Deletion of Statistics - This option will allow you to permenantly remove historical data about a link you own. This option is not reversable. If the link still exists when statistics are deleted, it effectively resets the counters the next time the link is accessed

### Link Metadata

A Link has the following information associated with it. Some parts are accessible depending on permissions and other data in here. This metadata is usually presented as various statistics

* Unique ID - This is managed by the API and can not be changed. Used for internal operations or referring to a specific link instance when making API calls
* User Unique ID - This is managed by the API and can not be changed (except by the API to gain ownership of deleted links). Used for internal operations or referring to a specific User. This is the owner of the link
* Long URL - This is the current URL that the short link will resolve to. Can be used to refer to the link, but may not guarentee any particular instance (the API optionally allows the same URL to be shortened multiple times)
* Short ID - This is the couple character code that is a part of the short URL. This is managed by the API and can not be changed. Can be used to refer to a specific link instance, but only if the link is owned by you
* Expiry Timer - This is a [UNIX Epoch Timestamp](https://en.wikipedia.org/wiki/Unix_time) that dictates when the API will no longer resolve a link; effectively deleting the link
* Protected - This is a user controllable character field that allows the creation of a password. When set, the link will not be resolved unless provided with the matching password. This field is not hashed and should not be considered to have the same security or protections as a user or application
* Remember Protected Accessors - Boolean. If someone accesses the link and enters the correct password once, a cookie will be stored on their machine that will allow them to access the link again without re-entering the password. User controllable, boolean
* Creation Time - This readonly field provides the UNIX Epoch Timestamp of the API Server when a link was created
* Update Time - This readonly field provides the UNIX Epoch Timestamp of the API Server when any link metadata was changed
* Deletion Time - This readonly field provides the UNIX Epoch Timestamp of the API Server when a link was deleted, or 0 when a link has not been deleted
* Metadata Deletion Time - This readonly field provides the UNIX Epoch Timestamp of the API Server when the counters were reset
* Created By - This readonly field provides the UUID of the User (or Application) that created the link
* Updated By - This readonly field provides the UUID of the User (or Application) that most recently changed any link metadata
* Deleted By - This readonly field provides the UUID of the User (or Application) that deleted the link
* Metadata Deleted By - This readonly field provides the UUID of the User (or Application) that reset link counters
* Delta - This readonly field stores the previous value metadata fields that were updated across all changes, time and user/application, except for readonly fields
* Total Clicks - This readonly field provides the total amount of clicks on the short link, calls to Resolve a link are counted here as well
* Unique Clicks - This readonly field stores the User Agent of each browser that clicks a link, along with a unique ID (generated in lieu of the connecting IP, ie you can see each unique user agent from a single IP address without seeing the IP Address itself)
* Successful Protected Clicks - This readonly field provides the total amount of times the link password was sucessfully entered. When Remember Protected Accessors is True, their cookie-based clicks are included
* Unsuccessful Protected Clicks - This readonly field provides the total amount of times the link password was either not entered, or was entered wrongly
* Short Resolutions - This readonly field provides a running counter of how many times the API was called to resolve the Short URL
* Long Resolutions - This readonly field provides a running counter of how many times the API was called to resolve the Long URL
* Social Clicks - This readonly field provides a running counter of how many times any social media service accesses a link (in IM, this usually happens once per user a link is shared to, and as they mark the message as read)
* Social Headers - Boolean or Selection. When a Long URL is provided that has social media headers or html heading tags (usually page image for articles), those will be fetched and made available for selection. When enabled, the selected headers or html tags will be provided to services that access the URL (such as twitter, facebook, instant messenger) 

### User Metadata

A User is defined as an individual (or group of individuals that share the same login) that accesses features provided by the API. You can use the website (which is an Application of this API) without creating an account, but will not be able to use Features such as detailed statistics, updating links, and deleting links. When a User is deleted, this is a one-way process and can not be "Un-deleted" like links can; A Deleted User's links will become owned by the API (no more modification, deletion, or detailed statistics), while their Applications will become deleted. 

* Unique ID - This is managed by the API and can not be changed. Used for internal operations or referring to a specific user when making API calls
* Username - This is the user's email address. It is used to uniquely identify a user, ie. there can only be one account per email address. This can be changed at any time by the user
* Password Hash - This is the result of a cryptographic operation that combines a Salt and the user provided password so that the password can not be easily derived in case of a database leak. This field is never returned in any API calls, but can be updated by the user at any time. When empty, the user must go through the password reset process (one time link)
* Applications - A list of the Applications that are registered to a user. Each can be managed individually, or some options can be adjusted in bulk, or individually deleted
* Links - A list of the Links that are registered to a user. Each can be managed individually. If multiple users register the same link, they will have the option of using the existing link. When that option is chosen, only the first user that registered an unexpired link will be able to make changes to or delete links
* Notes - This is a list of notes that are created on a User, either by the user themselves or administration for two way communication and support
* Creation Time - This readonly field provides the UNIX Epoch Timestamp of the API Server when a user was created
* Update Time - This readonly field provides the UNIX Epoch Timestamp of the API Server when a user was updated
* Updated By - This readonly field provides the UUID of the User (or Application) that most recently changed any User Metadata
* Delta - This readonly feild stores the previous value of metadata fields that were updated across all changes, time and uuid of user/application, except for readonly fields

### Application Settings

When an Application is deleted, this is a one-way process and can not be "Un-deleted" like links can. An Application is registered by a User, and it will contain the following information:

* Unique ID - This is managed by the API and can not be changed. Used for internal operations or referring to a specifc Application when making API calls
* User Unique ID - This is managed by the API and can not be changed. Used for internal operations or referring to a specific User when making API calls. This is the user who owns this Application
* API Key - This is a unique string generated by the API that uniquely identifies your Application, even if it shares a name with another user's Application. There is an option to regenerate the Key
* Secret - This is a long unique string generated by the API that is treated in a similar fashion to a user password. This proves that the Application owns the API Key provided. There is an option to regenerate the Secret
* Name - This is a human readable name for your application. It is recommended, but not required to be a unique string to allow for easy identification when you have multiple Applications
* Description - Similar to the name, this is a human readable text description of your Application
* Permissions - Defined [below](#permissions), dictate which [Features](#features) the Application is allowed to use
* Rate Limits - Can be defined by the user, or will be administratively imposed if an Application is found to be causing significant load on the API
* Creation Time - This readonly field provides the UNIX Epoch Timestamp of the API Server when an Application was created
* Update Time - This readonly field provides the UNIX Epoch Timestamp of the API Server when an Application was updated
* Updated By - This readonly field provides the UUID of the User (or Application) that most recently changed any Application Settings
* Delta - This readonly field stores the previous value of settings fields that were updated across all changes, time and uuid of user/application, except readonly fields

#### Permissions

Applications each have specific permissions, stored under its [Application Settings](#application-settings). The available permissions are as follows:

* Shorten Links - Allows the Application to create new short links, or to reuse an existing short link
* Resolve Links - Allows the Application to get either the short link or the long link, the opposite of what is provided
* Update Links - Can be either Boolean or a Selection. When Boolean (True), the Application can update ALL links owned by your user. When a Selection, the Application can update ONLY the links provided in the Selection.
* Delete Links - Can be either Boolean or a Selection. When Boolean (True), the Application can delete ALL links owned by your user. When a Selection, the Application can delete ONLY the links provided in the Selection.
* Read Statistics - Can be either a Boolean or a Selection. When Boolean (True), the Application can get statistics of ALL links owned by your user, and any non-Protected links. When a Selection, the Application can get statistics ONLY for the links provided in the Selection. 
* Modify User - Allows changing of username and password, and requesting a one-time password reset link of the user that owns the Application
* Modify Application - Can be either a Boolean or a Selection. When Boolean (True), the Application can change ALL Applications, except for itself, and grant or remove permissions (equal to or lower than what this Application has. An Application can not give permissions it does not have) owned by your user. When a Selection, the Application can do the same changes as Boolean, but ONLY to the selected Applications
* Delete Application - Can be either a Boolean or a Selection. When Boolean (True), the Applicaition can delete ALL Applications, except itself. When a Selection, the Application can delete ONLY the selected Applications

## Authentication

There are 2 methods of authentication:

* Username/Password - Used by end [users](#user-metadata) or administrators; Allows access to a dashboard that shows details such as API calls, rate limits (if any), generation of new API Keys, setting specific-application permissions, and so on
* API Key/Secret - Used by [Applications](#application-settings) to perform operations on behalf of a User; Has various user-defined [permissions](#permissions) and user defined (optional administrator overridden) rate limits

Session cookies are provided upon successful authentication. These cookies contain metadata such as unique IDs and which permissions the user or application have. Note that changing any values contained within a session cookie will invalidate it and force authentication again.

## Endpoints

Everything is to be under the /v2/ path on the server that provides this API. When this API becomes effective, the / and /V1/ paths will remain for legacy readonly purposes. All but two requests must be authenticated and have an active session. The API is only available via HTTPS connections. The API communicates via JSON POSTed to each endpoint, with some endpoints returning data on GET

* `/v2/authenticate` - Returns a session cookie upon successful authentication, returns HTTP 401 otherwise
    * Type - Either Application or User
    * Username - API Key or User email address
    * Password - Secret or User password
* `/v2/reset` - GET request with the following parameters
    * Token - A unique string of alphanumeric characters to be completed once. Used to authenticate the request
    * User Unique ID - the UUID of the user who's password is getting reset
* `/v2/shorten` - Shortens the provided Long URL and returns the new Short ID. If the Long URL has been shortened by the user that owns the application (or any application owned by the user), it will always return the existing Short ID. If the Long URL has been shortened by any other user (or their owned applications), a confirmation prompt will be given that requests a new API call to be made
    * Session - Provide your authorization cookie
    * Long URL - This is the long URL that will be shortened
    * No Verify - Optional, Boolean. Set to True to not fail adding the Link if it can't be reached by the API's status checker
    * Generate New - Optional, Boolean. Set to True to generate a new Link instance (with new Short ID and counters). This only has effect when a Long URL has been registered by a different user. Each user is limited to one short link per unique Long URL (eg. https://example.com will always be 000 for User A, but https://example.com/1, https://example.com/?1, etc are different Long URLs. When User B shortens https://example.com with this flag set, they will get 001, and so on)
* `/v2/resolve` - Returns either the Short ID or the Long URL for any link, unless the link is protected and not owned by your user
    * Session - Provide your authorization cookie. If unauthenticated, you will see one or more Short IDs for all non-protected occurrences of the Long URL; If authenticated, and you don't provide a Truthy value for See Public, you will only see the Short ID for your Long URL
    * URL - Provide the Short ID (eg, abc), Full Short Link (eg, https://lob.li/abc or lob.li/abc), or full Long URL (eg, https://example.com or example.com)
    * See Public - Optional, Boolean. When set to True, any link that is not protected and matches the provided URL will return its Short ID
* `/v2/modify` - Container for modification endpoints
    * `/link`- Returns available Link Metadata, along with status of the API call
        * Session - Provide your authorization cookie. If you don't have permission, you will get an error and no metadata
        * Link Unique ID - The UUID of the link that is being modified, or the short id
        * Short ID - The short id of the link that is being modified, or the uuid
        * Long URL - Optional, Character. When you provide this field, and the Link has existed for at least 24 hours, the Long URL that the Short ID resolves to will change to the provided value, given successful status checks
        * No Verify - Optional, Boolean. Set this to True to not fail changing the provided Long URL if it can't be reached by the API's status checker. NOOP if Long URL is not set
        * Expiry - Optional, UNIX Epoch Timestamp for when the link automatically expires. Must be a time in the future longer than 30 seconds 
        * Protected - Optional, Character. When you provide this field, the link will start requiring users to enter the provided password to resolve the link
        * Remember Protected Accessors - Optional, Boolean. When this field is True, the link will start allowing users to only enter the password once (as long as they have the cookie on their machine). NOOP if the link is not already Protected, or if the Protected field is not provided
    * `/application` - Returns available Application Settings and Permissions, along with status of the API call
        * Session - Provide your authorization cookie. If you don't have permission, you will get an error and no metadata
        * Application Unique ID - UUID of the Application that is being changed, can not be the Application making the request
        * Permissions - Optional, JSON containing the available permissions keys and values. If any permission is set that the user/application does not currently have, the result will be a failure with HTTP 403
        * Rate Limit - Optional, Allows setting the Rate Limit setting, which is a key:value of requests per second, day, week, and month; all optional. Rate Limits affect ALL API calls that an application may make, except for authentication
        * Name - Optional, Character. Defines a human readable name for the Application
        * Description - Optional, Text. Defines a human readable description for the Application
        * Regenerate Key - Optional, Boolean. When True, a new Application Key and Secret will be generated and provided in the response
    * `/user` - Returns available User Metadata, along with the status of the API call
        * Session - Provide your authorization cookie. If you don't have permission, you will get an error and no metadata
        * User Unique ID - UUID of the user that is being changed, must be the user that owns this application
        * Username - Optional, Character. When set, the user's login username will be changed to this. The user will receive a notification email to both their previous email and new email
        * Password - Optional, Character. When set, the user's login password hash will be recalculated based on what was provided. The user will receive a notification email to their username to indicate that their password was changed
        * Reset Password - Optional, Boolean. When True, the user will receive a notification email to their username to ask them to reset their password
* `/v2/delete` - Container for deletion endpoints
    * `/link` - Returns the UNIX Epoch Timestamp when they can no-longer "Un-delete" the link, and success if the application has Delete Links permission, HTTP 403 otherwise
        * Session - Provide your authorization cookie
        * Unique Link ID - The UUID of the link that you want to delete. If the Application doesn't have permission for that specific link, the API will return HTTP 403
        * Short ID - The Short ID of the link that you want to delete. Provide either the Unique Link ID or the Short ID; providing both will only look at the FIRST field provided
        * Reason - Optional, Text. Anything provided here will be recorded as a Note on your user account
    * `/statistics` - Returns success if the Statistics have been successfully cleared, HTTP 403 otherwise, except for server error
        * Session - Provide your authorization cookie
        * Unique Link ID - The UUID of the link that you want to reset counters on
    * `/application` - Returns success if the Application has been sucessfully deleted, HTTP 403 otherwise, except for server error
        * Session - Provide your authorization cookie
        * Unique Application ID - The UUID of the Application that will be deleted, must be owned by the same user that owns this application, can't be this application
        * Reason - Optional, Text. Anything provided here will be recorded as a Note on your user account
* `/v2/statistics` - Returns the Link Metadata on one or more links if they share the same Long URL (if ID Type is Long URL) provided that the application has permissions or if the link is unprotected and owned by a different user; otherwise HTTP 403, except for server error
    * Session - Provide your authorization cookie. 
    * ID - Provide one of the following values: uuid (the Unique ID of the link), short (the Short ID of the link, either https://lob.li/abc or lob.li/abc or abc), long (the Long URL of the link, either https://example.com or example.com)
