FORMAT: 1A
HOST: http://demo.ckan.org/

# CKAN

CKAN's **Action API** is a powerful, RPC-style API that exposes all of CKAN's
core features to API clients. All of a CKAN website's core functionality
(everything you can do with the web interface and more) can be used by external
code that calls the CKAN API.  For example, using the CKAN API your app can:

* Get JSON-formatted lists of a site's datasets, groups or other CKAN objects:
    * http://demo.ckan.org/api/3/action/package_list
    * http://demo.ckan.org/api/3/action/group_list
    * http://demo.ckan.org/api/3/action/tag_list
* Get a full JSON representation of a dataset, resource or other object:
    * http://demo.ckan.org/api/3/action/package_show?id=adur_district_spending
    * http://demo.ckan.org/api/3/action/tag_show?id=gold
    * http://demo.ckan.org/api/3/action/group_show?id=data-explorer
* Search for packages or resources matching a query:
    * http://demo.ckan.org/api/3/action/package_search?q=spending
    * http://demo.ckan.org/api/3/action/resource_search?query=name:District%20Names
* Create, update and delete datasets, resources and other objects
* Get an activity stream of recently changed datasets on a site:
    * http://demo.ckan.org/api/3/action/recently_changed_packages_activity_list

> Note: CKAN's [FileStore](http://docs.ckan.org/en/latest/maintaining/filestore.html) and [DataStore](http://docs.ckan.org/en/latest/maintaining/datastore.html) have their own APIs, see:

## Making an API request

To call the CKAN API, post a JSON dictionary in an HTTP POST request to one of
CKAN's API URLs. The parameters for the API function should be given in the
JSON dictionary. CKAN will also return its response in a JSON dictionary.

One way to post a JSON dictionary to a URL is using the command-line HTTP
client [HTTPie](http://httpie.org/).  For example, to get a list of the names
of all the datasets in the ``data-explorer`` group on demo.ckan.org, install
HTTPie and then call the ``group_list`` API function by running this command
in a terminal:

   http http://demo.ckan.org/api/3/action/group_list id=data-explorer

The response from CKAN will look like this:

    {
        "help": "...",
        "result": [
            "data-explorer",
            "department-of-ricky",
            "geo-examples",
            "geothermal-data",
            "reykjavik",
            "skeenawild-conservation-trust"
        ],
        "success": true
    }

The response is a JSON dictionary with three keys:

1. ``"success"``: ``true`` or ``false``.

   The API aims to always return ``200 OK`` as the status code of its HTTP
   response, whether there were errors with the request or not, so it's
   important to always check the value of the ``"success"`` key in the response
   dictionary and (if success is ``false``) check the value of the ``"error"``
   key.

   If there are major formatting problems with a request to the API, CKAN
   may still return an HTTP response with a ``409``, ``400`` or ``500``
   status code (in increasing order of severity). In future CKAN versions
   we intend to remove these responses, and instead send a ``200 OK``
   response and use the ``"success"`` and ``"error"`` items.

2. ``"result"``: the returned result from the function you called. The type
   and value of the result depend on which function you called. In the case of
   the ``group_list`` function it's a list of strings, the names of all the
   datasets that belong to the group.

   If there was an error responding to your request, the dictionary will
   contain an ``"error"`` key with details of the error instead of the
   ``"result"`` key. A response dictionary containing an error will look like
   this:

    {
        "help": "Creates a package",
        "success": false,
        "error": {
            "message": "Access denied",
            "__type": "Authorization Error"
        }
    }

3. ``"help"``: the documentation string for the function you called.

The same HTTP request can be made using Python's standard ``urllib2`` module,
with this Python code

    #!/usr/bin/env python
    import urllib2
    import urllib
    import json
    import pprint

    # Use the json module to dump a dictionary to a string for posting.
    data_string = urllib.quote(json.dumps({'id': 'data-explorer'}))

    # Make the HTTP request.
    response = urllib2.urlopen('http://demo.ckan.org/api/3/action/group_list',
            data_string)
    assert response.code == 200

    # Use the json module to load CKAN's response into a dictionary.
    response_dict = json.loads(response.read())

    # Check the contents of the response.
    assert response_dict['success'] is True
    result = response_dict['result']
    pprint.pprint(result)


## Example: Importing datasets with the CKAN API

You can add datasets using CKAN's web interface, but when importing many
datasets it's usually more efficient to automate the process in some way.  In
this example, we'll show you how to use the CKAN API to write a Python script
to import datasets into CKAN.

    #!/usr/bin/env python
    import urllib2
    import urllib
    import json
    import pprint

    # Put the details of the dataset we're going to create into a dict.
    dataset_dict = {
        'name': 'my_dataset_name',
        'notes': 'A long description of my dataset',
    }

    # Use the json module to dump the dictionary to a string for posting.
    data_string = urllib.quote(json.dumps(dataset_dict))

    # We'll use the package_create function to create a new dataset.
    request = urllib2.Request(
        'http://www.my_ckan_site.com/api/action/package_create')

    # Creating a dataset requires an authorization header.
    # Replace *** with your API key, from your user account on the CKAN site
    # that you're creating the dataset on.
    request.add_header('Authorization', '***')

    # Make the HTTP request.
    response = urllib2.urlopen(request, data_string)
    assert response.code == 200

    # Use the json module to load CKAN's response into a dictionary.
    response_dict = json.loads(response.read())
    assert response_dict['success'] is True

    # package_create returns the created package as its result.
    created_package = response_dict['result']
    pprint.pprint(created_package)


## API versions

The CKAN APIs are versioned. If you make a request to an API URL without a
version number, CKAN will choose the latest version of the API::

    http://demo.ckan.org/api/action/package_list

Alternatively, you can specify the desired API version number in the URL that
you request:

    http://demo.ckan.org/api/3/action/package_list

Version 3 is currently the only version of the Action API.

We recommend that you specify the API number in your requests, because this
ensures that your API client will work across different sites running different
version of CKAN (and will keep working on the same sites, when those sites
upgrade to new versions of CKAN). Because the latest version of the API may
change when a site is upgraded to a new version of CKAN, or may differ on
different sites running different versions of CKAN, the result of an API
request that doesn't specify the API version number cannot be relied on.


## Authentication and API keys

Some API functions require authorization. The API uses the same authorization
functions and configuration as the web interface, so if a user is authorized to
do something in the web interface they'll be authorized to do it via the API as
well.

When calling an API function that requires authorization, you must authenticate
yourself by providing your API key with your HTTP request. To find your API
key, login to the CKAN site using its web interface and visit your user profile
page.

To provide your API key in an HTTP request, include it in either an
``Authorization`` or ``X-CKAN-API-Key`` header.  (The name of the HTTP header
can be configured with the ``apikey_header_name`` option in your CKAN
configuration file.)

For example, to ask whether or not you're currently following the user
``markw`` on demo.ckan.org using HTTPie, run this command:

    http http://demo.ckan.org/api/3/action/am_following_user id=markw Authorization:XXX

(Replacing ``XXX`` with your API key.)

Or, to get the list of activities from your user dashboard on demo.ckan.org,
run this Python code:

    request = urllib2.Request('http://demo.ckan.org/api/3/action/dashboard_activity_list')
    request.add_header('Authorization', 'XXX')
    response_dict = json.loads(urllib2.urlopen(request, '{}').read())


## GET-able API functions

Functions defined in `ckan.logic.action.get`_ can also be called with an HTTP
GET request.  For example, to get the list of datasets (packages) from
demo.ckan.org, open this URL in your browser:

http://demo.ckan.org/api/3/action/package_list

Or, to search for datasets (packages) matching the search query ``spending``,
on demo.ckan.org, open this URL in your browser:

http://demo.ckan.org/api/3/action/package_search?q=spending

> Tip: Browser plugins like [JSONView for Firefox](https://addons.mozilla.org/en-us/firefox/addon/jsonview/)
> or [Chrome](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc)
> will format and color CKAN's JSON response nicely in your browser.

The search query is given as a URL parameter ``?q=spending``. Multiple
URL parameters can be appended, separated by ``&`` characters, for example
to get only the first 10 matching datasets open this URL:

http://demo.ckan.org/api/3/action/package_search?q=spending&rows=10

When an action requires a list of strings as the value of a parameter, the
value can be sent by giving the parameter multiple times in the URL:

http://demo.ckan.org/api/3/action/term_translation_show?terms=russian&terms=romantic%20novel

## JSONP support

To cater for scripts from other sites that wish to access the API, the data can
be returned in JSONP format, where the JSON data is 'padded' with a function
call. The function is named in the 'callback' parameter. For example:

http://demo.ckan.org/api/3/action/package_show?id=adur_district_spending&callback=myfunction


## API Examples

### Uploading a new version of a resource file

You can use the ``upload`` parameter of the
`ckan.logic.action.update.resource_update` function to upload a
new version of a resource file. This requires a ``multipart/form-data``
request, with httpie you can do this using the ``@file.csv``:

    http --json POST http://demo.ckan.org/api/3/action/resource_update id=<resource id> upload=@updated_file.csv Authorization:<api key>


## Action API reference

If you call one of the action functions listed below and the function
raises an exception, the API will return a JSON dictionary with keys
``"success": false`` and and an ``"error"`` key indicating the exception
that was raised.

For example `ckan.logic.action.get.member_list` (which returns a list of the
members of a group) raises `ckan.logic.NotFound` if the group doesn't exist.
If you called it over the API, you'd get back a JSON dict like this:

    {
        "success": false
        "error": {
            "__type": "Not Found Error",
            "message": "Not found"
        },
        "help": "...",
    }

# Group Retrieval functions

API functions for searching for and getting data from CKAN.

## Site read [/api/3/action/site_read]

### Check if site can be read [GET]

+ Response 200 (application/json)

        {
            "help": "Return ``True``...",
            "success": true,
            "result": true
        }

## Package list [/api/3/action/package_list]

### Get a list of all packages [GET]

Return a list of the names of the site’s datasets (packages).

+ limit (int) – if given, the list of datasets will be broken into pages of at most limit datasets per page and only one page will be returned at a time (optional)
+ offset (int) – when limit is given, the offset to start returning packages from

+ Response 200 (application/json)

        {
            "help": "Return a list of the names of the site's datasets (packages)...",
            "success": true,
            "result": ["dataset_1_id", "dataset_2_id"]
        }

## Package list with resources [/api/3/action/current_package_list_with_resources]

### Get current list of packages with resources [GET]

Return a list of the site’s datasets (packages) and their resources.

The list is sorted most-recently-modified first.

+ limit (int) – if given, the list of datasets will be broken into pages of at most limit datasets per page and only one page will be returned at a time (optional)
+ offset (int) – when limit is given, the offset to start returning packages from
+ page (int) – when limit is given, which page to return, Deprecated: use offset

+ Response 200 (application/json)

        {
            "help": "Return a list of the site's datasets (packages) and their resources..."",
            "success": true,
            "result": [
                        {
                            "license_title": "Creative Commons Attribution",
                            "license_id": "cc-by",
                            "maintainer": "",
                            "maintainer_email": "",
                            "relationships_as_object": [],
                            "private": false,
                            "revision_timestamp": "2015-08-12T16:11:23.514772",
                            "id": "84bf30b4-a37b-2204-a7a2-ff9081333a7f",
                            "metadata_created": "2014-08-19T14:36:54.739276",
                            "metadata_modified": "2015-08-31T23:03:23.536994",
                            "author": "",
                            "author_email": "",
                            "state": "active",
                            "version": "",
                            "creator_user_id": "2204bf6e-1255-4add-b61a-aeff1e3aadbb",
                            "type": "dataset",
                            "resources": 
                                [
                                    {
                                        "resource_group_id": "4f25e6a8-54a5-4287-8b32-6c33c4bf841f",
                                        "cache_last_updated": null,
                                        "revision_timestamp": "2015-08-31T23:03:23.535334",
                                        "webstore_last_updated": null,
                                        "id": "a61ec359-7a4d-4c01-bbc0-082bc442f4e4",
                                        "size": null,
                                        "state": "active",
                                        "last_modified": null,
                                        "hash": "",
                                        "description": "Description of first resource"",
                                        "format": "CSV",
                                        "mimetype_inner": null,
                                        "url_type": "upload",
                                        "mimetype": null,
                                        "cache_url": null,
                                        "name": "first-resource-of-dataset",
                                        "created": "2014-08-19T14:50:22.602232",
                                        "url": "http://demo.ckan.org/dataset/84bf30b4-a37b-2204-a7a2-ff9081333a7f/resource/a61ec359-7a4d-4c01-bbc0-082bc442f4e4/download/resourcefile.csv",
                                        "webstore_url": null,
                                        "position": 0,
                                        "revision_id": "1ab92fef-acdc-44c7-af0a-bc3a34ca35fc",
                                        "resource_type": null
                                    },
                                    {
                                        "resource_group_id": "4f25e6a8-54a5-4287-8b32-6c33c4bf841f",
                                        "cache_last_updated": null,
                                        "revision_timestamp": "2015-08-31T23:03:23.535334",
                                        "webstore_last_updated": null,
                                        "id": "0e728faf-d672-41f0-8bdf-2fcd25630fe2",
                                        "size": null,
                                        "state": "active",
                                        "last_modified": null,
                                        "hash": "",
                                        "description": "Description of second resource",
                                        "format": "CSV",
                                        "mimetype_inner": null,
                                        "url_type": "upload",
                                        "mimetype": null,
                                        "cache_url": null,
                                        "name": "second-resource-of-dataset",
                                        "created": "2014-08-19T14:42:57.143776",
                                        "url": "http://demo.ckan.org/dataset/84bf30b4-a37b-2204-a7a2-ff9081333a7f/resource/0e728faf-d672-41f0-8bdf-2fcd25630fe2/download/resourcefile2.csv",
                                        "webstore_url": null,
                                        "position": 1,
                                        "revision_id": "1ab92fef-acdc-44c7-af0a-bc3a34ca35fc",
                                        "resource_type": null
                                    }
                                ]
                            "num_resources": 2,
                            "tags":
                                [
                                    {
                                        "vocabulary_id": null,
                                        "display_name": "agriculture",
                                        "name": "agriculture",
                                        "revision_timestamp": "2014-08-19T14:36:54.739276",
                                        "state": "active",
                                        "id": "ba5cd720-a15b-4504-b292-967ff7eb35a7"
                                    },
                                    {
                                        "vocabulary_id": null,
                                        "display_name": "transport",
                                        "name": "transport",
                                        "revision_timestamp": "2014-08-19T14:36:54.739276",
                                        "state": "active",
                                        "id": "22042fae-ecb5-46f3-a8a2-aa7534b337ad"
                                    }
                                ],
                            "num_tags": 2,
                            "groups": [],
                            "relationships_as_subject": [],
                            "organization":
                                {
                                    "description": "Description of organisation..."",
                                    "title": "Public organisation",
                                    "created": "2014-08-18T15:03:05.801431",
                                    "approval_status": "approved",
                                    "revision_timestamp": "2014-08-18T15:03:05.673889",
                                    "is_organization": true,
                                    "state": "active",
                                    "image_url": "22040818-152305.668514pblcbdy.jpg",
                                    "revision_id": "41da8af8-224f-4169-3a93-8afa9f40f789",
                                    "type": "organization",
                                    "id": "d3d817ae-f5ae-4b5c-8adf-3473f11c12af",
                                    "name": "public-org"
                                },
                            "name": "my-first-dataset",
                            "isopen": true,
                            "url": "",
                            "notes": "Description of dataset...",
                            "owner_org": "d3d817ae-f5ae-4b5c-8adf-3473f11c12af",
                            "extras": [],
                            "license_url": "http://www.opendefinition.org/licenses/cc-by",
                            "title": "My first dataset",
                            "revision_id": "0f5e2932-b57c-48ac-af05-22de3163e5a4"
                        }
                    ]
        }

## Revision list [/api/3/action/revision_list]

### Get the revision list for the site [GET]

Return a list of the IDs of the site’s revisions. They are sorted with the newest first.

Since the results are limited to 50 IDs, you can page through them using parameter since_id.

+ since_id – the revision ID after which you want the revisions
+ since_time – the timestamp after which you want the revisions
+ sort (string) – the order to sort the related items in, possible values are ‘time_asc’, ‘time_desc’ (default). (optional)

+ Response 200 (application/json)

        {
            "help": "Return a list of the IDs of the site's revisions...",
            "success": true,
            "result": [
                "90bf2ad8-5d2b-45b1-8320-5a6d47c6a425",
                "aab91f3f-aea3-4ac7-af0a-bffa39ccf309"
            ]
        }

## Package revision list [/api/3/action/package_revision_list]

### Get a package's revision list [GET]

Return a dataset (package)’s revisions as a list of dictionaries.

+ id (string) – the id or name of the dataset

+ Response 200 (application/json)

        {
            "help": "Return a dataset (package)'s revisions as a list of dictionaries...",
            "success": true,
            "result": [
                {
                    "id": "90bffa38-5d2b-4ab0-8420-5a6c4726d4dc",
                    "timestamp": "2015-08-31T23:03:35.309555",
                    "message": "REST API: Update object my-dataset",
                    "author": "alice",
                    "approved_timestamp": null
                },
                {
                    "id": "13fbd4e3-a04c-4ea8-b822-b82304c69eef",
                    "timestamp": "2015-08-12T17:06:13.184237",
                    "message": "REST API: Update object my-dataset",
                    "author": "bob",
                    "approved_timestamp": null
                }
            ]
        }

## Group members [/api/3/action/member_list]

### Get the members of a group [GET]

Return the members of a group.

The user must have permission to ‘get’ the group.

+ id (string) – the id or name of the group
+ object_type (string) – restrict the members returned to those of a given type, e.g. 'user' or 'package' (optional, default: None)
+ capacity (string) – restrict the members returned to those with a given capacity, e.g. 'member', 'editor', 'admin', 'public', 'private' (optional, default: None)

+ Response 200 (application/json)

        {
            "help": "Return the members of a group...",
            "success": true,
            "result": [
                [
                    "d941e5af-2fb8-47fe-a297-3068f12331c2", 
                    "package", 
                    "public"
                ], 
                [
                    "c1952224-de21-45e9-90bd-d01c4a5c75c9", 
                    "package", 
                    "public"
                ]
            ]
        }

## Group list [/api/3/action/group_list]

### Get group list [GET]

Return a list of the names of the site’s groups.

+ order_by (string) – the field to sort the list by, must be 'name' or 'packages' (optional, default: 'name') Deprecated use sort.
+ sort (string) – sorting of the search results. Optional. Default: “name asc” string of field name and sort-order. The allowed fields are ‘name’, ‘package_count’ and ‘title’
+ limit (int) – if given, the list of groups will be broken into pages of at most limit groups per page and only one page will be returned at a time (optional)
+ offset (int) – when limit is given, the offset to start returning groups from
+ groups (list of strings) – a list of names of the groups to return, if given only groups whose names are in this list will be returned (optional)
+ all_fields (boolean) – return group dictionaries instead of just names. Only core fields are returned - get some more using the include_* options. Returning a list of packages is too expensive, so the packages property for each group is deprecated, but there is a count of the packages in the package_count property. (optional, default: False)
+ include_extras (boolean) – if all_fields, include the group extra fields (optional, default: False)
+ include_tags (boolean) – if all_fields, include the group tags (optional, default: False)
+ include_groups (boolean) – if all_fields, include the groups the groups are in (optional, default: False).
+ include_users (boolean) – if all_fields, include the group users (optional, default: False).

+ Response 200 (application/json)

        {
            "help": "Return a list of the names of the site's groups...",
            "success" true,
            "result": [
                "a-group",
                "b-group"
            ]
        }

## Organization list [/api/3/action/organization_list]

### Get list of organizations [GET]

Return a list of the names of the site’s organizations.

+ order_by (string) – the field to sort the list by, must be 'name' or 'packages' (optional, default: 'name') Deprecated use sort.
+ sort (string) – sorting of the search results. Optional. Default: “name asc” string of field name and sort-order. The allowed fields are ‘name’, ‘package_count’ and ‘title’
+ limit (int) – if given, the list of organizations will be broken into pages of at most limit organizations per page and only one page will be returned at a time (optional)
+ offset (int) – when limit is given, the offset to start returning organizations from
+ organizations (list of strings) – a list of names of the groups to return, if given only groups whose names are in this list will be returned (optional)
+ all_fields (boolean) – return group dictionaries instead of just names. Only core fields are returned - get some more using the include_* options. Returning a list of packages is too expensive, so the packages property for each group is deprecated, but there is a count of the packages in the package_count property. (optional, default: False)
+ include_extras (boolean) – if all_fields, include the organization extra fields (optional, default: False)
+ include_tags (boolean) – if all_fields, include the organization tags (optional, default: False)
+ include_groups – if all_fields, include the organizations the organizations are in (optional, default: False)
+ include_users (boolean) – if all_fields, include the organization users (optional, default: False).

+ Response 200 (application/json)

        {
            "help": "Return a list of the names of the site's organizations...",
            "success" true,
            "result": [
                "first-public-body",
                "another-public-body"
            ]
        }

## Group edit authorizations [/api/3/action/group_list_authz]

### Get list of groups user can edit [GET]

Return the list of groups that the user is authorized to edit.

+ available_only (boolean) – remove the existing groups in the package (optional, default: False)
+ am_member – if True return only the groups the logged-in user is a member of, otherwise return all groups that the user is authorized to edit (for example, sysadmin users are authorized to edit all groups) (optional, default: False)

+ Response 200 (application/json)

        {
            "help": "Return the list of groups that the user is authorized to edit...", 
            "success": true,
            "result": [
                {
                    "approval_status": "approved", 
                    "description": "Description of group.", 
                    "display_name": "A collection of random datasets", 
                    "id": "cd0a8d74-dab7-4d08-903f-ef29dfa824dd", 
                    "image_display_url": "", 
                    "image_url": "", 
                    "is_organization": false, 
                    "name": "random-datasets", 
                    "packages": 8, 
                    "revision_id": "21a43458-e60f-46b0-f660-efa89c20c1b4", 
                    "state": "active", 
                    "title": "A collection of random datasets", 
                    "type": "group"
                }
            ]
        }

## User's organizations list [/api/3/action/organization_list_for_user]

### Get organization list the user has a permission for [GET]

Return the organizations that the user has a given permission for.

By default this returns the list of organizations that the currently authorized user can edit, i.e. the list of organizations that the user is an admin of.

Specifically it returns the list of organizations that the currently authorized user has a given permission (for example: “manage_group”) against.

When a user becomes a member of an organization in CKAN they’re given a “capacity” (sometimes called a “role”), for example “member”, “editor” or “admin”.

Each of these roles has certain permissions associated with it. For example the admin role has the “admin” permission (which means they have permission to do anything). The editor role has permissions like “create_dataset”, “update_dataset” and “delete_dataset”. The member role has the “read” permission.

This function returns the list of organizations that the authorized user has a given permission for. For example the list of organizations that the user is an admin of, or the list of organizations that the user can create datasets in. This takes account of when permissions cascade down an organization hierarchy.

+ permission (string) – the permission the user has against the returned organizations, for example "read" or "create_dataset" (optional, default: "edit_group")

+ Response 200 (application/json)

        {
            "help": "Return the list of organizations that the user is a member of...",
            "success": true,
            "result": [
                {
                    "approval_status": "approved", 
                    "description": "Description of organization", 
                    "display_name": "An organization", 
                    "id": "a0d817f7-f5ae-4b5c-8adf-d472011c4fa4", 
                    "image_display_url": "http://demo.ckan.org/uploads/group/20140818-150305.668514pblcbdy.jpg", 
                    "image_url": "20140818-150305.668514pblcbdy.jpg", 
                    "is_organization": true, 
                    "name": "an-organisation", 
                    "packages": 3, 
                    "revision_id": "22dd89f8-213f-4169-ac93-8a0aef45e731", 
                    "state": "active", 
                    "title": "An organization", 
                    "type": "organization"
                }
            ]
        }

## Group revision list [/api/3/action/group_revision_list]

### Get list of revisions for a group [GET]

Return a group’s revisions.

+ id (string) – the name or id of the group
    
+ Response 200 (application/json)


        {
            "help": "Return a group's revisions...", 
            "result": [
                {
                    "approved_timestamp": null, 
                    "author": "alice", 
                    "id": "ebd2163e-64d2-4d8d-899c-d2bb7eb102db", 
                    "message": "REST API: Create member object ", 
                    "timestamp": "2015-08-12T10:05:46.868635"
                }, 
                {
                    "approved_timestamp": null, 
                    "author": "bob", 
                    "id": "71e2362f-1c7a-4c82-af85-aa60d7066d7c", 
                    "message": "REST API: Create object my-group", 
                    "timestamp": "2015-08-12T10:05:46.123168"
                }
            ], 
            "success": true
        }


## Organization revision list [/api/3/action/organization_revision_list]

### Get list of revisions for an organization [GET]

Return an organization’s revisions.

+ id (string) – the name or id of the organization

+ Response 200 (application/json)

        {
            "help": "Return an organization's revisions...", 
            "result": [
                {
                    "approved_timestamp": null, 
                    "author": "alice", 
                    "id": "c175d1bf-fa02-4bdf-adc0-d11cb4cc7277", 
                    "message": "", 
                    "timestamp": "2015-09-08T10:52:58.131248"
                }, 
                {
                    "approved_timestamp": null, 
                    "author": "bob", 
                    "id": "a2f9c871-45a6-4321-bab2-301cd9deab50", 
                    "message": "", 
                    "timestamp": "2015-08-12T14:26:24.577000"
                }
            ], 
            "success": true
        }

## License list [/api/3/action/license_list]

### Get list of licenses available to datasets on the site [GET]

Return the list of licenses available for datasets on the site.

+ Response 200 (application/json)

        {
            "help": "Return the list of licenses available for datasets on the site...", 
            "result": [
                {
                    "domain_content": "False", 
                    "domain_data": "False", 
                    "domain_software": "False", 
                    "family": "", 
                    "id": "notspecified", 
                    "is_generic": "True", 
                    "is_okd_compliant": false, 
                    "is_osi_compliant": false, 
                    "maintainer": "", 
                    "od_conformance": "not reviewed", 
                    "osd_conformance": "not reviewed", 
                    "status": "active", 
                    "title": "License not specified", 
                    "url": ""
                }, 
                {
                    "domain_content": "False", 
                            "domain_data": "True", 
                    "domain_software": "False", 
                    "family": "", 
                    "id": "odc-pddl", 
                    "is_generic": "False", 
                    "is_okd_compliant": true, 
                    "is_osi_compliant": false, 
                    "maintainer": "", 
                    "od_conformance": "approved", 
                    "osd_conformance": "not reviewed", 
                    "status": "active", 
                    "title": "Open Data Commons Public Domain Dedication and License (PDDL)", 
                    "url": "http://www.opendefinition.org/licenses/odc-pddl"
                }, 
                {
                    "domain_content": "False", 
                    "domain_data": "True", 
                    "domain_software": "False", 
                    "family": "", 
                    "id": "odc-odbl", 
                    "is_generic": "False", 
                    "is_okd_compliant": true, 
                    "is_osi_compliant": false, 
                    "maintainer": "", 
                    "od_conformance": "approved", 
                    "osd_conformance": "not reviewed", 
                    "status": "active", 
                    "title": "Open Data Commons Open Database License (ODbL)", 
                    "url": "http://www.opendefinition.org/licenses/odc-odbl"
                }
            ],
            "success": true
        }


## Tag list [/api/3/action/tag_list]

### Get list of tags used on the site [GET]

Return a list of the site’s tags.

By default only free tags (tags that don’t belong to a vocabulary) are returned. If the vocabulary_id argument is given then only tags belonging to that vocabulary will be returned instead.

+ query (string) – a tag name query to search for, if given only tags whose names contain this string will be returned (optional)
+ vocabulary_id (string) – the id or name of a vocabulary, if give only tags that belong to this vocabulary will be returned (optional)
+ all_fields (boolean) – return full tag dictionaries instead of just names (optional, default: False)

+ Response 200 (application/json)

        {
            "help": "Return a list of the site's tags...", 
            "result": [
                "USA", 
                "africa", 
                "aid", 
                "bandwidth", 
                "credit", 
                "date-2009", 
                "economics", 
                "election", 
                "geocoded", 
                "gold", 
                "housing", 
                "mortgages", 
                "occupy", 
                "poll", 
                "regional", 
                "spending", 
                "time-series", 
                "transparency"
            ], 
            "success": true
        }

## User list [/api/3/action/user_list]

### Get list of users on the site [GET]

Return a list of the site’s user accounts.

User properties include: number_of_edits which counts the revisions by the user and number_created_packages which excludes datasets which are private or draft state.

+ q (string) – restrict the users returned to those whose names contain a string (optional)
+ order_by (string) – which field to sort the list by (optional, default: 'name'). Can be any user field or edits (i.e. number_of_edits).

+ Response 200 (application/json)

        {
            "help": "Return a list of the site's user accounts...", 
            "result": [
                {
                    "about": null, 
                    "activity_streams_email_notifications": false, 
                    "created": "2015-09-08T12:36:24.851984", 
                    "display_name": "alice", 
                    "email_hash": "c160f8cc69a4f0bf2b0362752353d060", 
                    "fullname": null, 
                    "id": "997d86b6-61b0-4257-96c0-a9a10c0edeb5", 
                    "name": "alice", 
                    "number_created_packages": 0, 
                    "number_of_edits": 0, 
                    "openid": null, 
                    "state": "active", 
                    "sysadmin": false
                }, 
                {
                    "about": null, 
                    "activity_streams_email_notifications": false, 
                    "created": "2015-08-12T11:04:52.489297", 
                    "display_name": "bob", 
                    "email_hash": "338613c686c7da2f6f0553d8a1bfaaf8", 
                    "fullname": "Bob Smith", 
                    "id": "026707b8-c26f-4721-990f-993027856008", 
                    "name": "bob", 
                    "number_created_packages": 8, 
                    "number_of_edits": 416, 
                    "openid": null, 
                    "state": "active", 
                    "sysadmin": true
                }, 
                {
                    "about": null, 
                    "activity_streams_email_notifications": false, 
                    "created": "2015-08-12T11:04:49.402029", 
                    "display_name": "default", 
                    "email_hash": "d41d8cd98f00b204e9800998ecf8427e", 
                    "fullname": null, 
                    "id": "aa882ab7-acd2-45d8-8fd8-26debf6896ca", 
                    "name": "default", 
                    "number_created_packages": 0, 
                    "number_of_edits": 0, 
                    "openid": null, 
                    "state": "active", 
                    "sysadmin": true
                }
            ], 
            "success": true
        }

## Package show [/api/3/action/package_show]

### Get metadata for a package and its resources [GET]

Return the metadata of a dataset (package) and its resources.

+ id (string) – the id or name of the dataset
+ use_default_schema (bool) – use default package schema instead of a custom schema defined with an IDatasetForm plugin (default: False)
+ include_tracking (bool) – add tracking information to dataset and resources (default: False)

+ Response 200 (application/json)

        {
            "help": "Return the metadata of a dataset (package) and its resources...", 
            "result": {
                "author": "Alice Smith", 
                "author_email": "", 
                "creator_user_id": "026607b8-c26f-4721-990f-993027856008", 
                "extras": [
                    {
                        "key": "currency", 
                        "value": "EUR"
                    }
                ], 
                "groups": [
                    {
                        "description": "Group description.", 
                        "display_name": "Group Name", 
                        "id": "9a00ca13-5996-4267-baab-4183d7389d76", 
                        "image_display_url": "http://farm8.staticflickr.com/7129/7041988029_411d985015_c.jpg", 
                        "name": "group-name", 
                        "title": "Group Name"
                    }
                ], 
                "id": "74dbf50a-d653-4762-8d47-920609d2024a", 
                "isopen": false, 
                "license_id": "notspecified", 
                "license_title": "License not specified", 
                "maintainer": "", 
                "maintainer_email": "", 
                "metadata_created": "2015-08-12T10:05:45.331909", 
                "metadata_modified": "2015-08-12T14:26:24.577613", 
                "name": "example-dataset", 
                "notes": "Notes about this dataset", 
                "num_resources": 2, 
                "num_tags": 0, 
                "organization": {
                    "approval_status": "approved", 
                    "created": "2015-08-12T15:25:59.491303", 
                    "description": "", 
                    "id": "418cbbec-b1fc-42e7-82d5-74ced59c8145", 
                    "image_url": "", 
                    "is_organization": true, 
                    "name": "my-organization", 
                    "revision_id": "561aa493-1b29-463b-aa63-5b4eddd419cb", 
                    "state": "active", 
                    "title": "My Organization", 
                    "type": "organization"
                }, 
                "owner_org": "418cbbec-b1fc-42e7-82d5-74ced59c8145", 
                "private": false, 
                "relationships_as_object": [], 
                "relationships_as_subject": [], 
                "resources": [
                    {
                        "cache_last_updated": null, 
                        "cache_url": null, 
                        "created": "2015-08-12T11:05:45.344415", 
                        "datastore_active": false, 
                        "description": "First Resource (CSV)", 
                        "format": "CSV", 
                        "hash": "", 
                        "id": "d81037dc-6fb8-46f9-a79a-2d447bc2b12b", 
                        "last_modified": null, 
                        "mimetype": "", 
                        "mimetype_inner": "", 
                        "name": "", 
                        "openspending_hint": "data", 
                        "package_id": "74dbf50a-d653-4762-8d47-920609d2024a", 
                        "position": 0, 
                        "resource_type": "file", 
                        "revision_id": "8db3c25a-e6ab-4046-873b-54d60098d878", 
                        "size": null, 
                        "state": "active", 
                        "url": "http://example.com/data.csv", 
                        "url_type": null, 
                        "webstore_last_updated": null, 
                        "webstore_url": null
                    }, 
                    {
                        "cache_last_updated": null, 
                        "cache_url": null, 
                        "created": "2015-08-12T11:05:45.344434", 
                        "datastore_active": false, 
                        "description": "Second Resource (JSON)", 
                        "format": "JSON", 
                        "hash": "", 
                        "id": "1b2b09d2-8673-45f4-8c79-0fb0cd896a11", 
                        "last_modified": null, 
                        "mimetype": "", 
                        "mimetype_inner": "", 
                        "name": "", 
                        "openspending_hint": "model", 
                        "package_id": "74dbf50a-d653-4762-8d47-920609d2024a", 
                        "position": 1, 
                        "resource_type": "file", 
                        "revision_id": "8db3c25a-e6ab-4046-873b-54d60098d878", 
                        "size": null, 
                        "state": "active", 
                        "url": "https://example.com/model.js", 
                        "url_type": null, 
                        "webstore_last_updated": null, 
                        "webstore_url": null
                    }
                ], 
                "revision_id": "a2f9c871-45a6-4321-bab2-301cd9deab50", 
                "state": "active", 
                "tags": [], 
                "title": "An Example Dataset with Two Resources", 
                "type": "dataset", 
                "url": "http://www.example.com", 
                "version": ""
            }, 
            "success": true
        }

## Resource show [/api/3/action/resource_show]

### Get metadata for a resource [GET]

Return the metadata of a resource.

+ id (string) – the id of the resource
+ include_tracking (bool) – add tracking information to dataset and resources (default: False)

+ Response 200 (application/json)

        {
            "help": "Return the metadata of a resource...", 
            "result": {
                "cache_last_updated": null, 
                "cache_url": null, 
                "created": "2015-08-12T11:05:45.344415", 
                "datastore_active": false, 
                "description": "An example resource", 
                "format": "CSV", 
                "hash": "", 
                "id": "d81037dc-6fb8-46f9-a79a-2d447bc2b12b", 
                "last_modified": null, 
                "mimetype": "", 
                "mimetype_inner": "", 
                "name": "", 
                "openspending_hint": "data", 
                "package_id": "74dbf50a-d653-4762-8d47-920609d2024a", 
                "position": 0, 
                "resource_type": "file", 
                "revision_id": "8db3c25a-e6ab-4046-873b-54d60098d878", 
                "size": null, 
                "state": "active", 
                "url": "http://example.com/data.csv", 
                "url_type": null, 
                "webstore_last_updated": null, 
                "webstore_url": null
            }, 
            "success": true
        }


## Resource View metadata [/api/3/action/resource_view_show]

### Get metadata for a Resource View [GET]

Return the metadata of a resource_view.

+ id (string) – the id of the resource_view

+ Response 200 (application/json)

        {
            "help": "Return the metadata of a resource_view...", 
            "result": {
                "description": "", 
                "id": "e5731187-88c2-4147-b7c7-5d298434859e", 
                "package_id": "74dbf50a-d653-4762-8d47-920609d2024a", 
                "resource_id": "d81037dc-6fb8-46f9-a79a-2d447bc2b12b", 
                "title": "Data Explorer", 
                "view_type": "recline_view"
            }, 
            "success": true
        }

## Resource View list [/api/3/action/resource_view_list]

## Get a list of resource views for a resource [GET]

Return the list of resource views for a particular resource.

+ id (string) – the id of the resource

+ Response 200 (application/json)

        {
            "help": "Return the list of resource views for a particular resource...", 
            "result": [
                {
                    "description": "", 
                    "id": "51f7912d-6ca6-4799-b463-455677dbe95d", 
                    "package_id": "7638447d-5a74-4da1-ac87-baa21bcae4c3", 
                    "resource_id": "b9aae52b-b082-4159-b46f-7bb9c158d013", 
                    "title": "Data Explorer", 
                    "view_type": "recline_view"
                }, 
                {
                    "description": "A short resource description.", 
                    "id": "00a49402-8e36-4778-94d2-b86edd58a209", 
                    "package_id": "7638447d-5a74-4da1-ac87-baa21bcae4c3", 
                    "resource_id": "b9aae52b-b082-4159-b46f-7bb9c158d013", 
                    "title": "A Data Grid View", 
                    "view_type": "recline_grid_view"
                }
            ], 
            "success": true
        }

## Revision show [/api/3/action/revision_show]

### Get the details for a revision [GET]

Return the details of a revision.

+ id (string) – the id of the revision

+ Response 200 (application/json)

        {
            "help": "Return the details of a revision...", 
            "result": {
                "approved_timestamp": null, 
                "author": "alice", 
                "groups": [], 
                "id": "dcda86af-4db5-4707-90da-17d5e24cf597", 
                "message": "REST API: Create object my_dataset", 
                "packages": [
                    "my_dataset"
                ], 
                "timestamp": "2015-08-12T10:05:43.660195"
            }, 
            "success": true
        }

## Group show [/api/3/action/group_show]

### Get the details for a group [GET]

Return the details of a group. Only its first 1000 datasets are returned.


+ id (boolean) – the id or name of the group
+ include_datasets – include a list of the group’s datasets (optional, default: False)
+ include_extras – include the group’s extra fields (optional, default: True)
+ include_users – include the group’s users (optional, default: True)
+ include_groups – include the group’s sub groups (optional, default: True)
+ include_tags – include the group’s tags (optional, default: True)
+ include_followers – include the group’s number of followers (optional, default: True)

+ Response 200 (application/json)

        {
            "help": "Return the details of a group...", 
            "result": {
                "approval_status": "approved", 
                "created": "2015-08-12T11:05:46.147228", 
                "description": "A short group description", 
                "display_name": "My Group", 
                "extras": [], 
                "groups": [], 
                "id": "9a00ca13-5996-4267-baab-4183d7389d76", 
                "image_display_url": "http://farm8.staticflickr.com/7129/7041988029_411d985015_c.jpg", 
                "image_url": "http://farm8.staticflickr.com/7129/7041988029_411d985015_c.jpg", 
                "is_organization": false, 
                "name": "my-group", 
                "num_followers": 0, 
                "package_count": 6, 
                "revision_id": "71e2362f-1c7a-4c82-af85-aa60d7066d7c", 
                "state": "active", 
                "tags": [], 
                "title": "My Group", 
                "type": "group", 
                "users": [
                    {
                        "about": null, 
                        "activity_streams_email_notifications": false, 
                        "capacity": "admin", 
                        "created": "2015-08-12T11:04:52.489297", 
                        "display_name": "alice", 
                        "email_hash": "338513c686c7da2f6f0553d8a0bfa3f8", 
                        "fullname": null, 
                        "id": "026707b8-c26f-4721-990f-993027856008", 
                        "name": "alice", 
                        "number_created_packages": 8, 
                        "number_of_edits": 416, 
                        "openid": null, 
                        "state": "active", 
                        "sysadmin": true
                    }
                ]
            }, 
            "success": true
        }


## Organization show [/api/3/action/organization_show]

### Get the details of an organization [GET]

Return the details of a organization. Only its first 1000 datasets are returned.

+ id (boolean) – the id or name of the organization
+ include_datasets – include a list of the organization’s datasets (optional, default: False)
+ include_extras – include the organization’s extra fields (optional, default: True)
+ include_users – include the organization’s users (optional, default: True)
+ include_groups – include the organization’s sub groups (optional, default: True)
+ include_tags – include the organization’s tags (optional, default: True)
+ include_followers – include the organization’s number of followers (optional, default: True)

+ Response 200 (application/json)

        {
            "help": "Return the details of a organization...", 
            "result": {
                "approval_status": "approved", 
                "created": "2015-08-12T15:25:59.491303", 
                "description": "", 
                "display_name": "My Organization", 
                "extras": [], 
                "groups": [], 
                "id": "418cbbec-b1fc-42e7-82d5-74ced59c8145", 
                "image_display_url": "", 
                "image_url": "", 
                "is_organization": true, 
                "name": "my-organization", 
                "num_followers": 0, 
                "package_count": 2, 
                "revision_id": "561aa493-1b29-463b-aa63-5b4eddd419cb", 
                "state": "active", 
                "tags": [], 
                "title": "My Organization", 
                "type": "organization", 
                "users": [
                    {
                        "about": null, 
                        "activity_streams_email_notifications": false, 
                        "capacity": "admin", 
                        "created": "2015-08-12T11:04:52.489297", 
                        "display_name": "alice", 
                        "email_hash": "338513c686c7da2f6f0553d8a0bfa3f8", 
                        "fullname": null, 
                        "id": "026707b8-c26f-4721-990f-993027856008", 
                        "name": "alice", 
                        "number_created_packages": 8, 
                        "number_of_edits": 416, 
                        "openid": null, 
                        "state": "active", 
                        "sysadmin": true
                    }
                ]
            }, 
            "success": true
        }


## Show a group's packages [/api/3/action/group_package_show]

### Get a list of a group's packages [GET]

Return the datasets (packages) of a group.

+ id (string) – the id or name of the group
+ limit (int) – the maximum number of datasets to return (optional)

+ Response 200 (application/json)

        {
            "help": "Return the datasets (packages) of a group...", 
            "result": [
                {
                    "author": "", 
                    "author_email": "", 
                    "creator_user_id": "026707b8-c26f-4721-990f-993027856008", 
                    "extras": [
                        {
                            "key": "spatial", 
                            "value": "{ \"type\": \"Polygon\", \"coordinates\": [ [ [74.898827, 29.394159],[74.898827, 38.453041], [60.50526, 38.453041], [60.50526, 29.394159], [74.898827, 29.394159] ] ] }"
                        }, 
                        {
                            "key": "spatial-text", 
                            "value": "Afghanistan"
                        }
                    ], 
                    "groups": [
                        {
                            "description": "A group description.", 
                            "display_name": "Data Explorer Examples", 
                            "id": "9a00ca13-5996-4267-baab-4183d7389d76", 
                            "image_display_url": "http://farm8.staticflickr.com/7129/7041988029_411d985015_c.jpg", 
                            "name": "data-explorer", 
                            "title": "Data Explorer Examples"
                        }
                    ], 
                    "id": "de276618-8ab0-4aae-b306-cf38ff3688a8", 
                    "isopen": false, 
                    "license_id": "notspecified", 
                    "license_title": "License not specified", 
                    "maintainer": "", 
                    "maintainer_email": "", 
                    "metadata_created": "2015-08-12T10:05:45.155433", 
                    "metadata_modified": "2015-09-08T10:52:58.135004", 
                    "name": "first-dataset", 
                    "notes": "Notes about the first dataset.", 
                    "num_resources": 1, 
                    "num_tags": 5, 
                    "organization": {
                        "approval_status": "approved", 
                        "created": "2015-08-12T15:25:59.491303", 
                        "description": "", 
                        "id": "418cbbec-b1fc-42e7-82d5-74ced59c8145", 
                        "image_url": "", 
                        "is_organization": true, 
                        "name": "my-organization", 
                        "revision_id": "561aa493-1b29-463b-aa63-5b4eddd419cb", 
                        "state": "active", 
                        "title": "My Organization", 
                        "type": "organization"
                    }, 
                    "owner_org": "418cbbec-b1fc-42e7-82d5-74ced59c8145", 
                    "private": false, 
                    "relationships_as_object": [], 
                    "relationships_as_subject": [], 
                    "resources": [
                        {
                            "cache_last_updated": null, 
                            "cache_url": null, 
                            "created": "2015-08-12T11:05:45.180889", 
                            "description": "An example resource", 
                            "format": "CSV", 
                            "hash": "", 
                            "id": "f6331f99-51f6-44d9-95b9-b20f3b74f360", 
                            "last_modified": null, 
                            "mimetype": "", 
                            "mimetype_inner": "", 
                            "name": "", 
                            "package_id": "de276618-8ab0-4aae-b306-cf38ff3688a8", 
                            "position": 0, 
                            "resource_type": "", 
                            "revision_id": "d5c6f63b-41d2-46ce-bb17-9eb1c3f13776", 
                            "size": null, 
                            "state": "active", 
                            "url": "http://example.com/data.csv", 
                            "url_type": null, 
                            "webstore_last_updated": null, 
                            "webstore_url": null
                        }
                    ], 
                    "revision_id": "c175d1bf-fa02-4bdf-adc0-d11cb4cc7277", 
                    "state": "active", 
                    "tags": [
                        {
                            "display_name": "election", 
                            "id": "a93e00e0-19eb-4f3f-8f18-d3005c70a476", 
                            "name": "election", 
                            "state": "active", 
                            "vocabulary_id": null
                        }, 
                        {
                            "display_name": "politics", 
                            "id": "87dae2d9-30ec-47f1-b87b-90d8ee0f1443", 
                            "name": "politics", 
                            "state": "active", 
                            "vocabulary_id": null
                        }, 
                    ], 
                    "title": "An Example Dataset", 
                    "type": "dataset", 
                    "url": "http://example.com/", 
                    "version": ""
                }
                    ], 
                    "revision_id": "a2f9c871-45a6-4321-bab2-301cd9deab50", 
                    "state": "active", 
                    "tags": [], 
                    "title": "Italian Regional Public Accounts", 
                    "type": "dataset", 
                    "url": "http://www.dps.tesoro.it/cpt/banca_dati_home.asp", 
                    "version": ""
                }
            ], 
            "success": true
        }

## Tag show [/api/3/action/tag_show]

### Get the details of a tag [GET]

Return the details of a tag and all its datasets.

+ id (string) – the name or id of the tag
+ vocabulary_id (string) – the id or name of the tag vocabulary that the tag is in - if it is not specified it will assume it is a free tag. (optional)
+ include_datasets (bool) – include a list of the tag’s datasets. (Up to a limit of 1000 - for more flexibility, use package_search - see package_search() for an example.) (optional, default: False)

+ Response 200 (application/json)

        {
            "help": "Return the details of a tag and all its dataset...", 
            "result": {
                "display_name": "politics", 
                "id": "87dae2d9-30ec-47f1-b87b-90d8ee0f1443", 
                "name": "politics", 
                "vocabulary_id": null
            }, 
            "success": true
        }

## User show [/api/3/action/user_show]

### Return a user account [GET]

Return a user account.

Either the id or the user_obj parameter must be given.

Includes email_hash, number_of_edits and number_created_packages (which excludes draft or private datasets unless it is the same user or sysadmin making the request). Excludes the password (hash) and reset_key. If it is the same user or a sysadmin requesting, the email and apikey are included.

+ id (string) – the id or name of the user (optional)
+ user_obj (user dictionary) – the user dictionary of the user (optional)
+ include_datasets (boolean) – Include a list of datasets the user has created. If it is the same user or a sysadmin requesting, it includes datasets that are draft or private. (optional, default:False, limit:50)
+ include_num_followers (boolean) – Include the number of followers the user has (optional, default:False)

+ Response 200 (application/json)

        {

            "help": "http://demo.ckan.org/api/3/action/help_show?name=user_show",
            "success": true,
            "result": {
                "openid": null,
                "about": null,
                "display_name": "Alice",
                "name": "alice",
                "created": "2015-07-22T14:19:01.766116",
                "email_hash": "b642b4217b34b1e8d3bd915fc65c4452",
                "sysadmin": false,
                "activity_streams_email_notifications": false,
                "state": "active",
                "number_of_edits": 5,
                "fullname": "test test",
                "id": "bd6862e0-9db3-45de-a18e-8ee5b39d82ba",
                "number_created_packages": 0
            }
        }

## Package autocompete [/api/3/action/package_autocomplete]

### Return a list of completions for packages names [GET]

Return a list of datasets (packages) that match a string.

Datasets with names or titles that contain the query string will be returned.

+ q (string) – the string to search for
+ limit (int) – the maximum number of resource formats to return (optional, default: 10)

+ Response 200 (application/json)

        {

            "help": "http://demo.ckan.org/api/3/action/help_show?name=package_autocomplete",
            "success": true,
            "result": [
                {
                    "match_field": "title",
                    "match_displayed": "Dataset A",
                    "name": "dataset-a",
                    "title": "Dataset_A"
                },
                {
                    "match_field": "name",
                    "match_displayed": "Dataset B",
                    "name": "dataset-b",
                    "title": "Dataset B"
                }
            ]
        }

## Format autocomplete [/api/3/action/format_autocomplete]

### Return a list of resource formats whose names contain a string. [GET]

+ q (string) – the string to search for
+ limit (int) – the maximum number of resource formats to return (optional, default: 5)

+ Response 200 (application/json)

        {

            "help": "http://demo.ckan.org/api/3/action/help_show?name=format_autocomplete",
            "success": true,
            "result" : [
                "csv",
                "cvs",
                "docx",
            ]
        }


## User autocomplete [/api/3/action/user_autocomplete]

### Return a list of user names that contain a string. [GET]

+ q (string) – the string to search for
+ limit (int) – the maximum number of user names to return (optional, default: 20)

+ Response 200 (application/json)

        {

            "help": "http://demo.ckan.org/api/3/action/help_show?name=user_autocomplete",
            "success": true,
            "result": [
                {

                    "fullname": "test",
                    "id": "cf0d2040-0fe1-42f9-bcaf-95c417c11147",
                    "name": "test"

                },
                {

                    "fullname": "ckantest",
                    "id": "079f23a2-0ca3-44eb-96b5-174de8131e3b",
                    "name": "ckantest"

                }
            ]
        }

## Organization autocomplete [/api/3/action/organization_autocomplete]

### Return a list of organization names that contain a string. [GET]

+ q (string) – the string to search for
+ limit (int) – the maximum number of organizations to return (optional, default: 20)

+ Response 200 (application/json)

        {

            "help": "http://demo.ckan.org/api/3/action/help_show?name=organization_autocomplete",
            "success": true,
            "result": [
                {

                    "title": "Organization1",
                    "id": "a2da3c39-3571-4332-adf5-b356a432e1c8",
                    "name": "organization1"

                },
                {

                    "title": "Ross' Test Organization",
                    "id": "0756f5fd-f718-4478-ad65-25fcfcd5518b",
                    "name": "rosstestorganization"

                }
            ]
        }


## Package search [/api/3/action/package_search]

### Searches for packages satisfying a given search criteria.

This action accepts solr search query parameters (details below), and returns a dictionary of results, including dictized datasets that match the search criteria, a search count and also facet information.

Solr Parameters:

For more in depth treatment of each paramter, please read the Solr Documentation.

This action accepts a subset of solr’s search query parameters:

+ q (string) – the solr query. Optional. Default: "*:*"
+ fq (string) – any filter queries to apply. Note: +site_id:{ckan_site_id} is added to this string prior to the query being executed.
+ sort (string) – sorting of the search results. Optional. Default: 'relevance asc, metadata_modified desc'. As per the solr documentation, this is a comma-separated string of field names and sort-orderings.
+ rows (int) – the number of matching rows to return.
+ start (int) – the offset in the complete result for where the set of returned datasets should begin.
+ facet (string) – whether to enable faceted results. Default: True.
+ facet.mincount (int) – the minimum counts for facet fields should be included in the results.
+ facet.limit (int) – the maximum number of values the facet fields return. A negative value means unlimited. This can be set instance-wide with the search.facets.limit config option. Default is 50.
+ facet.field (list of strings) – the fields to facet upon. Default empty. If empty, then the returned facet information is empty.
+ include_drafts (boolean) – if True, draft datasets will be included in the results. A user will only be returned their own draft datasets, and a sysadmin will be returned all draft datasets. Optional, the default is False.

The following advanced Solr parameters are supported as well. Note that some of these are only available on particular Solr versions. See Solr’s dismax and edismax documentation for further details on them:

qf, wt, bf, boost, tie, defType, mm

Examples:

q=flood datasets containing the word flood, floods or flooding fq=tags:economy datasets with the tag economy facet.field=["tags"] facet.limit=10 rows=0 top 10 tags

Limitations:

The full solr query language is not exposed, including.

fl The parameter that controls which fields are returned in the solr query cannot be changed. CKAN always returns the matched datasets as dictionary objects.

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=package_search",
            "success": true,
            "result": [
                {
                    "author": "Alice Smith", 
                    "author_email": "", 
                    "creator_user_id": "026607b8-c26f-4721-990f-993027856008", 
                    "extras": [
                        {
                            "key": "currency", 
                            "value": "EUR"
                        }
                    ], 
                    "groups": [
                        {
                            "description": "Group description.", 
                            "display_name": "Group Name", 
                            "id": "9a00ca13-5996-4267-baab-4183d7389d76", 
                            "image_display_url": "http://farm8.staticflickr.com/7129/7041988029_411d985015_c.jpg", 
                            "name": "group-name", 
                            "title": "Group Name"
                        }
                    ], 
                    "id": "74dbf50a-d653-4762-8d47-920609d2024a", 
                    "isopen": false, 
                    "license_id": "notspecified", 
                    "license_title": "License not specified", 
                    "maintainer": "", 
                    "maintainer_email": "", 
                    "metadata_created": "2015-08-12T10:05:45.331909", 
                    "metadata_modified": "2015-08-12T14:26:24.577613", 
                    "name": "example-dataset", 
                    "notes": "Notes about this dataset", 
                    "num_resources": 2, 
                    "num_tags": 0, 
                    "organization": {
                        "approval_status": "approved", 
                        "created": "2015-08-12T15:25:59.491303", 
                        "description": "", 
                        "id": "418cbbec-b1fc-42e7-82d5-74ced59c8145", 
                        "image_url": "", 
                        "is_organization": true, 
                        "name": "my-organization", 
                        "revision_id": "561aa493-1b29-463b-aa63-5b4eddd419cb", 
                        "state": "active", 
                        "title": "My Organization", 
                        "type": "organization"
                    }, 
                    "owner_org": "418cbbec-b1fc-42e7-82d5-74ced59c8145", 
                    "private": false, 
                    "relationships_as_object": [], 
                    "relationships_as_subject": [], 
                    "resources": [
                        {
                            "cache_last_updated": null, 
                            "cache_url": null, 
                            "created": "2015-08-12T11:05:45.344415", 
                            "datastore_active": false, 
                            "description": "First Resource (CSV)", 
                            "format": "CSV", 
                            "hash": "", 
                            "id": "d81037dc-6fb8-46f9-a79a-2d447bc2b12b", 
                            "last_modified": null, 
                            "mimetype": "", 
                            "mimetype_inner": "", 
                            "name": "", 
                            "openspending_hint": "data", 
                            "package_id": "74dbf50a-d653-4762-8d47-920609d2024a", 
                            "position": 0, 
                            "resource_type": "file", 
                            "revision_id": "8db3c25a-e6ab-4046-873b-54d60098d878", 
                            "size": null, 
                            "state": "active", 
                            "url": "http://example.com/data.csv", 
                            "url_type": null, 
                            "webstore_last_updated": null, 
                            "webstore_url": null
                        }, 
                        {
                            "cache_last_updated": null, 
                            "cache_url": null, 
                            "created": "2015-08-12T11:05:45.344434", 
                            "datastore_active": false, 
                            "description": "Second Resource (JSON)", 
                            "format": "JSON", 
                            "hash": "", 
                            "id": "1b2b09d2-8673-45f4-8c79-0fb0cd896a11", 
                            "last_modified": null, 
                            "mimetype": "", 
                            "mimetype_inner": "", 
                            "name": "", 
                            "openspending_hint": "model", 
                            "package_id": "74dbf50a-d653-4762-8d47-920609d2024a", 
                            "position": 1, 
                            "resource_type": "file", 
                            "revision_id": "8db3c25a-e6ab-4046-873b-54d60098d878", 
                            "size": null, 
                            "state": "active", 
                            "url": "https://example.com/model.js", 
                            "url_type": null, 
                            "webstore_last_updated": null, 
                            "webstore_url": null
                        }
                    ], 
                    "revision_id": "a2f9c871-45a6-4321-bab2-301cd9deab50", 
                    "state": "active", 
                    "tags": [], 
                    "title": "An Example Dataset with Two Resources", 
                    "type": "dataset", 
                    "url": "http://www.example.com", 
                    "version": ""
                } 
            ]
            "count": 1,
            "sort": "score desc, metadata_modified desc",
            "facets": { },
            "search_facets": { }
        }

##  Resource search [/api/3/action/resource_search]

### Searches for resources satisfying a given search criteria. [GET}

Returns a dictionary with 2 fields: count and results. The count field contains the total number of Resources found without the limit or query parameters having an effect. The results field is a list of dictized Resource objects.

The ‘query’ parameter is a required field. It is a string of the form {field}:{term} or a list of strings, each of the same form. Within each string, {field} is a field or extra field on the Resource domain object.

If {field} is "hash", then an attempt is made to match the {term} as a prefix of the Resource.hash field.

If {field} is an extra field, then an attempt is made to match against the extra fields stored against the Resource.

Note: The search is limited to search against extra fields declared in the config setting ckan.extra_resource_fields.

Note: Due to a Resource’s extra fields being stored as a json blob, the match is made against the json string representation. As such, false positives may occur:

If the search criteria is:

query = "field1:term1"

Then a json blob with the string representation of:

{"field1": "foo", "field2": "term1"}

will match the search criteria! This is a known short-coming of this approach.

All matches are made ignoring case; and apart from the "hash" field, a term matches if it is a substring of the field’s value.

Finally, when specifying more than one search criteria, the criteria are AND-ed together.

The order parameter is used to control the ordering of the results. Currently only ordering one field is available, and in ascending order only.

The fields parameter is deprecated as it is not compatible with calling this action with a GET request to the action API.

The context may contain a flag, search_query, which if True will make this action behave as if being used by the internal search api. ie - the results will not be dictized, and SearchErrors are thrown for bad search queries (rather than ValidationErrors).

+ query (string or list of strings of the form {field}:{term1}) – The search criteria. See above for description.
+ fields (dict of fields to search terms.) – Deprecated
+ order_by (string) – A field on the Resource model that orders the results.
+ offset (int) – Apply an offset to the query.
+ limit (int) – Apply a limit to the query.

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=resource_search",
            "success": true,
            "result": {
                "count": 1,
                "results": [
                    {
                        "cache_last_updated": null,
                        "package_id": "03f79cfe-73df-45d0-9568-e7754041887b",
                        "webstore_last_updated": "2015-09-10T02:33:51.226254",
                        "id": "e8b60a16-e7a3-419e-a662-fd12a2d35d10",
                        "size": null,
                        "state": "active",
                        "last_modified": "2015-09-10T02:33:48.263026",
                        "hash": "293a8d53227b2255215bb208bd551d6e",
                        "description": "test-description",
                        "format": "CSV",
                        "mimetype_inner": null,
                        "url_type": "upload",
                        "mimetype": null,
                        "cache_url": null,
                        "name": "ttest",
                        "created": "2015-09-10T02:33:48.306399",
                        "url": "http://demo.ckan.org/dataset/03f79cfe-73df-45d0-9568-e7754041887b/resource/e8b60a16-e7a3-419e-a662-fd12a2d35d10/download/test.csv",
                        "webstore_url": "active",
                        "position": 0,
                        "revision_id": "97392911-bb65-4e07-a109-c82b95965986",
                        "resource_type": null
                    }
                ]
            }
        }

## Tag search [/api/3/action/tag_search]

### Return a list of tags whose names contain a given string. [GET]

By default only free tags (tags that don’t belong to any vocabulary) are searched. If the vocabulary_id argument is given then only tags belonging to that vocabulary will be searched instead.

+ query (string or list of strings) – the string(s) to search for
+ vocabulary_id (string) – the id or name of the tag vocabulary to search in (optional)
+ fields (dictionary) – deprecated
+ limit (int) – the maximum number of tags to return
+ offset (int) – when limit is given, the offset to start returning tags from

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=tag_search",
            "success": true,
            "result": {
                "count": 2,
                "results": [
                    {
                        "vocabulary_id": null,
                        "id": "94454a52-4287-4dc3-917f-0a676c259531",
                        "name": "test"
                    },
                    {
                        "vocabulary_id": null,
                        "id": "27de6814-2bcd-48fc-bb93-f3fdbac1b8a2",
                        "name": "testy"
                    }
                ]
            }
        }

## Tag autocomplete [/api/3/action/tag_autocomplete]

### Return a list of tag names that contain a given string. [GET]

By default only free tags (tags that don’t belong to any vocabulary) are searched. If the vocabulary_id argument is given then only tags belonging to that vocabulary will be searched instead.

+ query (string) – the string to search for
+ vocabulary_id (string) – the id or name of the tag vocabulary to search in (optional)
+ fields (dictionary) – deprecated
+ limit (int) – the maximum number of tags to return
+ offset (int) – when limit is given, the offset to start returning tags from

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=tag_autocomplete",
            "success": true,
            "result": [
                "test",
                "Test",
                "test1",
            ]
        }

## Task status show [/api/3/action/task_status_show]

### Return a task status. [GET]

Either the id parameter or the entity_id, task_type and key parameters must be given.

+ id (string) – the id of the task status (optional)
+ entity_id (string) – the entity_id of the task status (optional)
+ task_type – the task_type of the task status (optional)
+ key (string) – the key of the task status (optional)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=task_status_show",
            "success": true,
            "result": {
                "entity_id": "f6331f99-51f6-44d9-95b9-b20f3b74f360",
                "task_type": "datapusher",
                "last_updated": "2015-07-22T14:30:33.363745",
                "entity_type": "resource",
                "value": "6fb14c09-6f1e-477f-990c-aec12b0f67f7",
                "state": "complete",
                "key": "datapusher",
                "error": "{}",
                "id": "9f70c368-11eb-48d3-af63-c6d17821b942"
            }
        }

## Term translation show [/api/3/action/term_translation_show]

### Return the translations for the given term(s) and language(s). [GET]

Return the translations for the given term(s) and language(s).

returns a list of term translation dictionaries each with keys 'term' (the term searched for, in the source language), 'term_translation' (the translation of the term into the target language) and 'lang_code' (the language code of the target language)

+ terms (list of strings) – the terms to search for translations of, e.g. 'Russian', 'romantic novel'
+ lang_codes (list of language code strings) – the language codes of the languages to search for translations into, e.g. 'en', 'de' (optional, default is to search for translations into any language)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=term_translation_show",
            "success": true,
            "result": [
                {
                    "term": "test"
                    "term_translation": "test"
                    "lang_code": "de"
                }
             ]
        }

## Status show [/api/3/action/status_show]

### Show information about the site configuration [GET]

Return a dictionary with information about the site’s configuration.

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=status_show",
            "success": true,
            "result": {
            "ckan_version": "2.4.0",
            "site_url": "http://demo.ckan.org",
            "site_description": "Demo",
            "site_title": "CKAN",
            "error_emails_to": null,
            "locale_default": "en",
            "extensions": [
                    "stats",
                    "demo",
                    "datastore",
                    "resource_proxy",
                    "recline_view",
                    "text_view",
                    "pdf_view",
                    "repo_info",
                    "spatial_metadata",
                    "spatial_query",
                    "geojson_view",
                    "geo_view",
                    "datapusher",
                    "apihelper",
                    "image_view",
                    "recline_map_view",
                    "recline_graph_view",
                    "recline_grid_view",
                ]
            }
        }

## Vocabulary list  [/api/3/action/vocabulary_list]

### Return a list of all the site’s tag vocabularies. [GET]

Return a list of all the site’s tag vocabularies.

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=vocabulary_list",
                "success": true,
                "result": [
                    {
                    "tags": [
                        {
                            "vocabulary_id": "c6d5ef9d-045c-41b8-bbff-af6b495978ca",
                            "display_name": "de",
                            "id": "2c8dad54-db60-4eba-aa46-4bd543b40b37",
                            "name": "de"
                        },
                        {
                            "vocabulary_id": "c6d5ef9d-045c-41b8-bbff-af6b495978ca",
                            "display_name": "fr",
                            "id": "e54edd2a-cdec-46cb-bb4c-d78bf28b8583",
                            "name": "fr"
                        },
                        {
                            "vocabulary_id": "c6d5ef9d-045c-41b8-bbff-af6b495978ca",
                            "display_name": "ie",
                            "id": "547a3916-6dbc-4fa3-93cc-fc3c58226d61",
                            "name": "ie"
                        },
                        {
                            "vocabulary_id": "c6d5ef9d-045c-41b8-bbff-af6b495978ca",
                            "display_name": "uk",
                            "id": "b63bb4b9-2b86-461a-a462-1c30a898eb0e",
                            "name": "uk"
                        }
                    ],
                    "id": "c6d5ef9d-045c-41b8-bbff-af6b495978ca",
                    "name": "country_codes"
                }
            ]

        }

## Vocabulary show  [/api/3/action/vocabulary_show]

### Return a single tag vocabulary. [GET]

Return a single tag vocabulary

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=vocabulary_show",
            "success": true,
            "result": {
            "tags": [
                {
                    "vocabulary_id": "c6d5ef9d-045c-41b8-bbff-af6b495978ca",
                    "display_name": "de",
                    "id": "2c8dad54-db60-4eba-aa46-4bd543b40b37",
                    "name": "de"
                },
                {
                    "vocabulary_id": "c6d5ef9d-045c-41b8-bbff-af6b495978ca",
                    "display_name": "fr",
                    "id": "e54edd2a-cdec-46cb-bb4c-d78bf28b8583",
                    "name": "fr"
                },
                {
                    "vocabulary_id": "c6d5ef9d-045c-41b8-bbff-af6b495978ca",
                    "display_name": "ie",
                    "id": "547a3916-6dbc-4fa3-93cc-fc3c58226d61",
                    "name": "ie"
                },
                {
                    "vocabulary_id": "c6d5ef9d-045c-41b8-bbff-af6b495978ca",
                    "display_name": "uk",
                    "id": "b63bb4b9-2b86-461a-a462-1c30a898eb0e",
                    "name": "uk"
                }
            ],
                "id": "c6d5ef9d-045c-41b8-bbff-af6b495978ca",
                "name": "country_codes"
            }
        }
## User activity list [/api/3/action/user_activity_list]

### Return a user’s public activity stream. [GET]

Return a user’s public activity stream.
You must be authorized to view the user’s profile.

+ id (string) – the id or name of the user
+ offset (int) – where to start getting activity items from (optional, default: 0)
+ limit (int) – the maximum number of activities to return (optional, default: 31, the default value is configurable via the ckan.activity_list_limit setting)

+ Response 200 (application/json)

        {

            "help": "http://demo.ckan.org/api/3/action/help_show?name=user_activity_list",
            "success": true,
            "result": [
                {
                    "user_id": "7add58a7-236a-4131-9c4c-83e3cda869e0",
                    "timestamp": "2015-08-31T18:40:25.223720",
                    "object_id": "0add5867-296a-4131-9c4c-83e3cda869e0",
                    "revision_id": "fb85b0c4-dd65-47ee-a01d-5b710ba101ea",
                    "data": { },
                    "id": "aec9e262-ade5-4ac8-9426-d35d5140f011",
                    "activity_type": "new user"
                }
            ]

        }

## Package activity list [/api/3/action/package_activity_list]

### Return a package’s activity stream. [GET]

Return a package’s activity stream. You must be authorized to view the package.

+ id (string) – the id or name of the package
+ offset (int) – where to start getting activity items from (optional, default: 0)
+ limit (int) – the maximum number of activities to return (optional, default: 31, the default value is configurable via the ckan.activity_list_limit setting)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=package_activity_list",
            "success": true,
            "result": [
                {
                    "user_id": "01b3756a-e1ca-4d4a-b8f1-6880a00095d6",
                    "timestamp": "2015-07-22T14:30:19.412987",
                    "object_id": "7e4d4ef3-f452-4c35-963d-9c6e582374b3",
                    "revision_id": "0f85d5c5-d105-4586-a762-80805e42f7f8",
                    "data": {
                        "package": {
                            "maintainer": "",
                            "name": "adur_district_spending",
                            "metadata_modified": "2015-07-22T14:30:18.584924",
                            "author": "Lucy Chambers",
                            "url": "http://www.spotlightonspend.org.uk/Downloads/1038",
                            "notes": "From Spikes Cavell, Spotlight on Spend. \r\n\r\nFor Ardur, records from April 2009-March 2010 are currently available (2011-008-04) ",
                            "owner_org": null,
                            "private": false,
                            "maintainer_email": "",
                            "author_email": "",
                            "state": "active",
                            "version": "",
                            "creator_user_id": "01b3756a-e1ca-4d4a-b8f1-6880a00095d6",
                            "id": "7e4d4ef3-f452-4c35-963d-9c6e582374b3",
                            "title": "UK: Adur District Council Spending Data",
                            "revision_id": "bde1269e-65f2-40dc-8e33-a0e5390f7192",
                            "type": "dataset",
                            "license_id": "notspecified"
                        }
                    },
                    "id": "83d7fdbc-9c9d-4616-85fd-106a5b5d8425",
                    "activity_type": "changed package"

                },
                {

                    "user_id": "01b3756a-e1ca-4d4a-b8f1-6880a00095d6",
                    "timestamp": "2015-07-22T14:29:59.609118",
                    "object_id": "7e4d4ef3-f452-4c35-963d-9c6e582374b3",
                    "revision_id": "bde1269e-65f2-40dc-8e33-a0e5390f7192",
                    "data": {
                        "package": {
                            "maintainer": "",
                            "name": "adur_district_spending",
                            "metadata_modified": "2015-07-22T14:29:55.571155",
                            "author": "Lucy Chambers",
                            "url": "http://www.spotlightonspend.org.uk/Downloads/1038",
                            "notes": "From Spikes Cavell, Spotlight on Spend. \r\n\r\nFor Ardur, records from April 2009-March 2010 are currently available (2011-008-04) ",
                            "owner_org": null,
                            "private": false,
                            "maintainer_email": "",
                            "author_email": "",
                            "state": "active",
                            "version": "",
                            "creator_user_id": "01b3756a-e1ca-4d4a-b8f1-6880a00095d6",
                            "id": "7e4d4ef3-f452-4c35-963d-9c6e582374b3",
                            "title": "UK: Adur District Council Spending Data",
                            "revision_id": "bde1269e-65f2-40dc-8e33-a0e5390f7192",
                            "type": "dataset",
                            "license_id": "notspecified"
                        }
                    },
                    "id": "8bfd9289-9d80-4803-97e2-f16e86f5f49c",
                    "activity_type": "new package"
                }
            ]
        }

## Group activity list [/api/3/action/group_activity_list]

### Return a group’s activity stream. [GET]

Return a group’s activity stream. You must be authorized to view the group.

+ id (string) – the id or name of the group
+ offset (int) – where to start getting activity items from (optional, default: 0)
+ limit (int) – the maximum number of activities to return (optional, default: 31, the default value is configurable via the ckan.activity_list_limit setting)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=group_activity_list",
            "success": true,
            "result": [
                {
                    "user_id": "c6f6b0d0-f70c-481c-8be5-4ffd637615ee",
                    "timestamp": "2012-11-19T13:02:33.561904",
                    "object_id": "e4f345c5-5fc9-4767-9ed2-74b3c4ea2bc1",
                    "revision_id": "f131e6f9-e990-4e7d-a277-9b374ba64b41",
                    "data": {
                        "package": {
                            "maintainer": "",
                            "name": "adur_district_spending",
                            "title": "UK: Adur District Council Spending Data",
                            "url": "http://www.spotlightonspend.org.uk/Downloads/1038",
                            "notes": "From Spikes Cavell, Spotlight on Spend. \r\n\r\nFor Ardur, records from April 2009-March 2010 are currently available (2011-008-04) ",
                            "author": "Lucy Chambers",
                            "maintainer_email": "",
                            "author_email": "",
                            "state": "active",
                            "version": "",
                            "license_id": "notspecified",
                            "revision_id": "f131e6f9-e990-4e7d-a277-9b374ba64b41",
                            "type": null,
                            "id": "e4f345c5-5fc9-4767-9ed2-74b3c4ea2bc1"
                        }
                    },
                    "id": "dfba119f-ae45-4bd6-bad4-37de2a08c43e",
                    "activity_type": "new package"
                }
            ]
        }


## Organization activity list [/api/3/action/organization_activity_ilst]

### Return a organization’s activity stream. [GET]

Return a organization’s activity stream.

+ id (string) – the id or name of the organization

+ Response 200 (application/json)

        {
            "help": "http://master.ckan.org/api/3/action/help_show?name=organization_activity_list",
            "success": true,
            "result": [
                {
                    "user_id": "4405e682-c6b7-43ba-9489-f8ed8394658b",
                    "timestamp": "2013-11-28T09:52:10.270861",
                    "object_id": "3b368fb6-e3cc-4381-af80-5dd7e35cbaf6",
                    "revision_id": "b2fa366e-8c98-408e-bd99-ba70b1f086f3",
                    "data": {
                        "group": {
                            "description": "just a little test org",
                            "created": "2013-11-28T09:52:10.044127",
                            "title": "Test organization",
                            "name": "test-organization",
                            "is_organization": true,
                            "state": "active",
                            "image_url": "",
                            "revision_id": "b2fa366e-8c98-408e-bd99-ba70b1f086f3",
                            "type": "organization",
                            "id": "3b368fb6-e3cc-4381-af80-5dd7e35cbaf6",
                            "approval_status": "approved"
                        }
                    },
                    "id": "6a8bcf86-1236-4e88-a905-af194d94749d",
                    "activity_type": "new organization"
                }
            ]
        }
## Recently chagned packages activity list [/api/3/action/recently_changed_packages_activity_list]

### Return the activity stream of all recently added or changed packages. [GET]

+ offset (int) – where to start getting activity items from (optional, default: 0)
+ limit (int) – the maximum number of activities to return (optional, default: 31, the default value is configurable via the ckan.activity_list_limit setting)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=recently_changed_packages_activity_list",
            "success": true,
            "result": [
                {
                    "user_id": "cb37c133-1e1d-4325-87cf-013786677a9c",
                    "timestamp": "2015-09-29T15:32:56.738226",
                    "object_id": "d02a4462-aee3-4d25-befa-a1624ee1de75",
                    "revision_id": "1771b86f-8dcd-44fc-8814-1902f3638cb5",
                    "data": {
                        "package": {
                            "maintainer": null,
                            "name": "test",
                            "metadata_modified": "2015-09-29T15:32:55.787128",
                            "author": null,
                            "url": null,
                            "notes": "Dataset created with Python",
                            "owner_org": null,
                            "private": false,
                            "maintainer_email": null,
                            "author_email": null,
                            "state": "active",
                            "version": null,
                            "creator_user_id": "cb37c133-1e1d-4325-87cf-013786677a9c",
                            "id": "d02a4462-aee3-4d25-befa-a1624ee1de75",
                            "title": "test102",
                            "revision_id": "5cf89b20-b9ac-482c-9440-5ddcbb145662",
                            "type": "dataset",
                            "license_id": null
                        }
                    },
                    "id": "d7a7ed3e-e5ba-4f5c-b174-3ab3d923d21c",
                    "activity_type": "changed package"
                }
            ]
        }

## Activity detail list [/api/3/action/activity_detail_list]

### Return an activity’s list of activity detail items. [GET] 

Return an activity’s list of activity detail items.
+ id (string) – the id of the activity

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=activity_detail_list",
            "success": true,
            "result": [
                {
                    "activity_id": "d7a7ed3e-e5ba-4f5c-b174-3ab3d923d21c",
                    "object_type": "Resource",
                    "object_id": "aa43879b-e08f-40ff-8960-6a8d3b687906",
                    "data": {
                        "resource": {
                            "cache_last_updated": null,
                            "package_id": "d02a4462-aee3-4d25-befa-a1624ee1de75",
                            "webstore_last_updated": null,
                            "id": "aa43879b-e08f-40ff-8960-6a8d3b687906",
                            "size": null,
                            "state": "active",
                            "extras": { },
                            "hash": "",
                            "description": "test data",
                            "format": "CSV",
                            "mimetype_inner": null,
                            "url_type": "upload",
                            "mimetype": null,
                            "cache_url": null,
                            "name": "test",
                            "created": "2015-09-29T15:32:55.829498",
                            "url": "test.csv",
                            "webstore_url": null,
                            "last_modified": "2015-09-29T15:32:55.767227",
                            "position": 2,
                            "revision_id": "1771b86f-8dcd-44fc-8814-1902f3638cb5",
                            "resource_type": null
                        }
                    },
                    "id": "1d3871fa-b29d-496f-beb9-7b25ac145019",
                    "activity_type": "new"

                },
                {
                    "activity_id": "d7a7ed3e-e5ba-4f5c-b174-3ab3d923d21c",
                    "object_type": "Resource",
                    "object_id": "fb071dc7-4817-4f51-b453-a10b0cb4f1d7",
                    "data": {
                        "resource": {
                            "cache_last_updated": null,
                            "package_id": "d02a4462-aee3-4d25-befa-a1624ee1de75",
                            "webstore_last_updated": null,
                            "id": "fb071dc7-4817-4f51-b453-a10b0cb4f1d7",
                            "size": null,
                            "state": "active",
                            "extras": { },
                            "hash": "",
                            "description": "test",
                            "format": "csv",
                            "mimetype_inner": null,
                            "url_type": "upload",
                            "mimetype": null,
                            "cache_url": null,
                            "name": "test",
                            "created": "2015-09-29T14:10:34.069647",
                            "url": "http://demo.ckan.org/dataset/d02a4462-aee3-4d25-befa-a1624ee1de75/resource/fb071dc7-4817-4f51-b453-a10b0cb4f1d7/download/test.csv",
                            "webstore_url": null,
                            "last_modified": "2015-09-29T14:10:33.958064",
                            "position": 1,
                            "revision_id": "1771b86f-8dcd-44fc-8814-1902f3638cb5",
                            "resource_type": null
                        }
                    },
                    "id": "c199ac08-a5e5-4457-877c-9e3678f08ebc",
                    "activity_type": "changed"
                }
            ]
        }

## User activity list [/api/3/action/user_activity_list_html]

### Return a user’s public activity stream as HTML. [GET]

Return a user’s public activity stream as HTML.

The activity stream is rendered as a snippet of HTML meant to be included in an HTML page, i.e. it doesn’t have any HTML header or footer.

+ id (string) – The id or name of the user.
+ offset (int) – where to start getting activity items from (optional, default: 0)
+ limit (int) – the maximum number of activities to return (optional, default: 31, the default value is configurable via the ckan.activity_list_limit setting)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=user_activity_list_html",
            "success": true,
            "result": "\n\n\n\n  \n    <ul class=\"activity\" data-module=\"activity-stream\" data-module-more=\"False\" data-module-context=\"user\" data-module-id=\"joe\" data-module-offset=\"0\">\n      \n        \n        \n          \n            \n<!-- Snippet snippets/activity_item.html start -->\n<li class=\"item new-user\">\n  \n  <i class=\"icon icon-user\"></i>\n  <p>\n    <span class=\"actor\"><img src=\"//gravatar.com/avatar/f5b8fb60c6116331da07c65b96a8a1d1?s=30&amp;d=identicon\"\n        class=\"gravatar\" width=\"30\" height=\"30\" /> <a href=\"/user/joe\">joe</a></span> signed up\n    <span class=\"date\" title=\"August 31, 2015, 18:40\">28 days ago</span>\n  </p>\n</li>\n<!-- Snippet snippets/activity_item.html end -->\n\n          \n        \n        \n      \n    </ul>\n  \n"
        }


## Package activity list html [/api/3/action/package_activity_list_html]

### Return a package’s activity stream as HTML. [GET]

Return a package’s activity stream as HTML.

The activity stream is rendered as a snippet of HTML meant to be included in an HTML page, i.e. it doesn’t have any HTML header or footer.

+ id (string) – the id or name of the package
+ offset (int) – where to start getting activity items from (optional, default: 0)
+ limit (int) – the maximum number of activities to return (optional, default: 31, the default value is configurable via the ckan.activity_list_limit setting)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=package_activity_list_html",
            "success": true,
            "result": "\n\n\n\n  \n    <ul class=\"activity\" data-module=\"activity-stream\" data-module-more=\"False\" data-module-context=\"package\" data-module-id=\"adur_district_spending\" data-module-offset=\"0\">\n      \n        \n        \n          \n            \n<!-- Snippet snippets/activity_item.html start -->\n<li class=\"item changed-resource\">\n  \n  <i class=\"icon icon-file\"></i>\n  <p>\n    <span class=\"actor\"><img src=\"//gravatar.com/avatar/d41d8cd98f00b204e9800998ecf8427e?s=30&amp;d=identicon\"\n        class=\"gravatar\" width=\"30\" height=\"30\" /> <a href=\"/user/okfn\">okfn</a></span> updated the resource <a href=\"/dataset/7e4d4ef3-f452-4c35-963d-9c6e582374b3/resource/281dffa6-ea9b-4446-be41-05dced06591f\">Adur District Council April 2009</a> in the dataset <span><a href=\"/dataset/adur_district_spending\">UK: Adur District Council Spending Data</a></span>\n    <span class=\"date\" title=\"July 22, 2015, 14:30\">2 months ago</span>\n  </p>\n</li>\n<!-- Snippet snippets/activity_item.html end -->\n\n          \n        \n          \n            \n<!-- Snippet snippets/activity_item.html start -->\n<li class=\"item changed-resource\">\n  \n  <i class=\"icon icon-file\"></i>\n  <p>\n    <span class=\"actor\"><img src=\"//gravatar.com/avatar/d41d8cd98f00b204e9800998ecf8427e?s=30&amp;d=identicon\"\n        class=\"gravatar\" width=\"30\" height=\"30\" /> <a href=\"/user/okfn\">okfn</a></span> updated the resource <a href=\"/dataset/7e4d4ef3-f452-4c35-963d-9c6e582374b3/resource/04127ad5-77e5-4a08-9f40-12d3c383e460\">Revised CSV for import</a> in the dataset <span><a href=\"/dataset/adur_district_spending\">UK: Adur District Council Spending Data</a></span>\n    <span class=\"date\" title=\"July 22, 2015, 14:30\">2 months ago</span>\n  </p>\n</li>\n<!-- Snippet snippets/activity_item.html end -->\n\n          \n        \n          \n            \n<!-- Snippet snippets/activity_item.html start -->\n<li class=\"item new-package\">\n  \n  <i class=\"icon icon-sitemap\"></i>\n  <p>\n    <span class=\"actor\"><img src=\"//gravatar.com/avatar/d41d8cd98f00b204e9800998ecf8427e?s=30&amp;d=identicon\"\n        class=\"gravatar\" width=\"30\" height=\"30\" /> <a href=\"/user/okfn\">okfn</a></span> created the dataset <span><a href=\"/dataset/adur_district_spending\">UK: Adur District Council Spending Data</a></span>\n    <span class=\"date\" title=\"July 22, 2015, 14:29\">2 months ago</span>\n  </p>\n</li>\n<!-- Snippet snippets/activity_item.html end -->\n\n          \n        \n        \n      \n    </ul>\n  \n"
        }

## Group activity list html [/api/3/action/group_activity_list_html]

### Return a group’s activity stream as HTML. [GET]

Return a group’s activity stream as HTML.

The activity stream is rendered as a snippet of HTML meant to be included in an HTML page, i.e. it doesn’t have any HTML header or footer.

+ id (string) – the id or name of the group
+ offset (int) – where to start getting activity items from (optional, default: 0)
+ limit (int) – the maximum number of activities to return (optional, default: 31, the default value is configurable via the ckan.activity_list_limit setting)

+ Response 200 (application/json)

        {

            "help": "http://master.ckan.org/api/3/action/help_show?name=group_activity_list_html",
            "success": true,
            "result": "\n\n\n\n  \n    <ul class=\"activity\" data-module=\"activity-stream\" data-module-more=\"False\" data-module-context=\"group\" data-module-id=\"testtest\" data-module-offset=\"0\">\n      \n        \n        \n          \n            \n<!-- Snippet snippets/activity_item.html start -->\n<li class=\"item new-group\">\n  \n  <i class=\"icon icon-group\"></i>\n  <p>\n    <span class=\"actor\"><img src=\"//gravatar.com/avatar/d41d8cd98f00b204e9800998ecf8427e?s=30&amp;d=identicon\"\n        class=\"gravatar\" width=\"30\" height=\"30\" /> <a href=\"/user/admin\">admin</a></span> created the group <span><a href=\"/group/testtest\">testtest</a></span>\n    <span class=\"date\" title=\"February 6, 2015, 14:10 (UTC)\">7 months ago</span>\n  </p>\n</li>\n<!-- Snippet snippets/activity_item.html end -->\n\n          \n        \n        \n      \n    </ul>\n  \n"

        }

## Organization activity list html [/api/3/action/organization_activity_list_html]

### Return a organization’s activity stream as HTML. [GET]

Return a organization’s activity stream as HTML.

The activity stream is rendered as a snippet of HTML meant to be included in an HTML page, i.e. it doesn’t have any HTML header or footer.

+ id (string) – the id or name of the organization

+ Response 200 (application/json)

        {

            "help": "http://demo.ckan.org/api/3/action/help_show?name=organization_activity_list_html",
            "success": true,
            "result": "\n\n\n\n  \n    <ul class=\"activity\" data-module=\"activity-stream\" data-module-more=\"False\" data-module-context=\"organization\" data-module-id=\"vehicles\" data-module-offset=\"0\">\n      \n        \n        \n          \n            \n<!-- Snippet snippets/activity_item.html start -->\n<li class=\"item new-package\">\n  \n  <i class=\"icon icon-sitemap\"></i>\n  <p>\n    <span class=\"actor\"><img src=\"//gravatar.com/avatar/31dfd3857f3c68d3329a0b0fd9d6a8a6?s=30&amp;d=identicon\"\n        class=\"gravatar\" width=\"30\" height=\"30\" /> <a href=\"/user/pasquy73\">Pasquale Vitale</a></span> created the dataset <span><a href=\"/dataset/vehicles_4wheels\">vehicles_4wheels</a></span>\n    <span class=\"date\" title=\"September 24, 2015, 14:11\">5 days ago</span>\n  </p>\n</li>\n<!-- Snippet snippets/activity_item.html end -->\n\n          \n        \n          \n            \n<!-- Snippet snippets/activity_item.html start -->\n<li class=\"item new-organization\">\n  \n  <i class=\"icon icon-briefcase\"></i>\n  <p>\n    <span class=\"actor\"><img src=\"//gravatar.com/avatar/31dfd3857f3c68d3329a0b0fd9d6a8a6?s=30&amp;d=identicon\"\n        class=\"gravatar\" width=\"30\" height=\"30\" /> <a href=\"/user/pasquy73\">Pasquale Vitale</a></span> created the organization <a href=\"/organization/vehicles\">vehicles</a>\n    <span class=\"date\" title=\"September 24, 2015, 14:11\">5 days ago</span>\n  </p>\n</li>\n<!-- Snippet snippets/activity_item.html end -->\n\n          \n        \n        \n      \n    </ul>\n  \n"

        }


## Recently changed packages activity list html [/api/3/action/recently_changed_packages_activity_list_html]

### Return the activity stream of all recently changed packages as HTML. [GET]
Return the activity stream of all recently changed packages as HTML.

The activity stream includes all recently added or changed packages. It is rendered as a snippet of HTML meant to be included in an HTML page, i.e. it doesn’t have any HTML header or footer.

+ offset (int) – where to start getting activity items from (optional, default: 0)
+ limit (int) – the maximum number of activities to return (optional, default: 31, the default value is configurable via the ckan.activity_list_limit setting)

+ Response 200 (application/json)

        {

            "help": "http://demo.ckan.org/api/3/action/help_show?name=recently_changed_packages_activity_list_html",
            "success": true,
            "result": "\n\n\n\n  \n    <ul class=\"activity\" data-module=\"activity-stream\" data-module-more=\"False\" data-module-context=\"package\" data-module-id=\"\" data-module-offset=\"0\">\n      \n        \n        \n          \n            \n<!-- Snippet snippets/activity_item.html start -->\n<li class=\"item changed-package\">\n  \n  <i class=\"icon icon-sitemap\"></i>\n  <p>\n    <span class=\"actor\"><img src=\"//gravatar.com/avatar/fe63f2b226b156342b3afe67effdff1d?s=30&amp;d=identicon\"\n        class=\"gravatar\" width=\"30\" height=\"30\" /> <a href=\"/user/frank\">Frank Storm</a></span> updated the dataset <span><a href=\"/dataset/test102\">test102</a></span>\n    <span class=\"date\" title=\"September 29, 2015, 15:32\">2 hours ago</span>\n  </p>\n</li>\n<!-- Snippet snippets/activity_item.html end -->\n\n          \n        \n        \n      \n    </ul>\n  \n"

        }

## User follower count [/api/3/action/user_follower_count]

### Return the number of followers of a user. [GET]

Return the number of followers of a user.

+ id (string) – the id or name of the user

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=user_follower_count",
            "success": true,
            "result": 0
        }

## Dataset follower count [/api/3/action/dataset_follower_count]

### Return the number of followers of a dataset. [GET]

Return the number of followers of a dataset.

+ id (string) – the id or name of the dataset

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=dataset_follower_count",
            "success": true,
            "result": 0
        }

## Group follower count [/api/3/action/group_follower_count]

### Return the number of followers of a group. [GET]

Return the number of followers of a group.

+ id (string) – the id or name of the group

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=group_follower_count",
            "success": true,
            "result": 0
        }

## Organization follower count [/api/3/action/organization_follower_count]

### Return the number of followers of an organization. [GET]

Return the number of followers of an organization.

+ id (string) – the id or name of the organization

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=organization_follower_count",
            "success": true,
            "result": 0
        }
## User follower list [/api/3/action/user_follower_list]

### Return the list of users that are following the given user. [GET]

Return the list of users that are following the given user.
+ id (string) – the id or name of the user

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=user_follower_list",
            "success": true,
            "result": [
                {
                    "openid": null,
                    "about": null,
                    "display_name": "alice",
                    "name": "alice",
                    "created": "2015-09-29T18:45:57.067359",
                    "email_hash": "4b9411a9b28f9063ea75e5fee24bb2a8",
                    "sysadmin": false,
                    "activity_streams_email_notifications": false,
                    "state": "active",
                    "number_of_edits": 0,
                    "fullname": "alice",
                    "id": "abe5ac41-f035-484b-a358-c34088961c1c",
                    "number_created_packages": 0
                }
            ]
        }

## Dataset follower list [/api/3/action/dataset_follower_list]

### Return the list of users that are following the given dataset. [GET]

Return the list of users that are following the given dataset.

+ id (string) – the id or name of the dataset

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=dataset_follower_list",
            "success": true,
            "result": [
                {
                    "openid": null,
                    "about": null,
                    "display_name": "alice",
                    "name": "alice",
                    "created": "2015-09-29T18:53:15.415553",
                    "email_hash": "c8d86ad5ed1add319e802f7f659df166",
                    "sysadmin": true,
                    "activity_streams_email_notifications": false,
                    "state": "active",
                    "number_of_edits": 0,
                    "fullname": "test",
                    "id": "3d413bb0-0487-49fc-afe7-f5065b8aa683",
                    "number_created_packages": 0
                }
            ]
        }

## Group follower list [/api/3/action/group_follower_list]

### Return the list of users that are following the given group. [GET]

Return the list of users that are following the given group.

+ id (string) – the id or name of the group

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=group_follower_list",
            "success": true,
            "result": [
                {
                    "openid": null,
                    "about": null,
                    "display_name": "alice",
                    "name": "alice",
                    "created": "2015-09-29T18:53:15.415553",
                    "email_hash": "c8d86ad5ed1add319e802f7f659df166",
                    "sysadmin": true,
                    "activity_streams_email_notifications": false,
                    "state": "active",
                    "number_of_edits": 0,
                    "fullname": "test",
                    "id": "3d413bb0-0487-49fc-afe7-f5065b8aa683",
                    "number_created_packages": 0
                }
            ]
        }

## Organization follower list [/api/3/action/organization_follower_list]

### Return the list of users that are following the given organization. [GET]

Return the list of users that are following the given organization.

+ id (string) – the id or name of the organization

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=organization_follower_list",
            "success": true,
            "result": [
                {
                    "openid": null,
                    "about": null,
                    "display_name": "alice",
                    "name": "alice",
                    "created": "2015-09-29T18:53:15.415553",
                    "email_hash": "c8d86ad5ed1add319e802f7f659df166",
                    "sysadmin": true,
                    "activity_streams_email_notifications": false,
                    "state": "active",
                    "number_of_edits": 0,
                    "fullname": "test",
                    "id": "3d413bb0-0487-49fc-afe7-f5065b8aa683",
                    "number_created_packages": 0
                }
            ]
        }


## Am following user [/api/3/action/am_following_user]

### Return True if you’re following the given user, False if not. [GET]

Return True if you’re following the given user, False if not.

+ id (string) – the id or name of the user

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=am_following_user",
            "success": true,
            "result": false
        }

## Am following dataset [/api/3/action/am_following_dataset]

### Return True if you’re following the given dataset, False if not. [GET]

Return True if you’re following the given dataset, False if not.

+ id (string) – the id or name of the dataset

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=am_following_dataset",
            "success": true,
            "result": true
        }

## Am following group [/api/3/action/am_following_group]

### Return True if you’re following the given group, False if not. [GET]

Return True if you’re following the given group, False if not.

+ id (string) – the id or name of the group

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=am_following_group",
            "success": true,
            "result": true
        }

## Followee count [/api/3/action/followee_count]

### Return the number of objects that are followed by the given user. [GET]

Return the number of objects that are followed by the given user.

Counts all objects, of any type, that the given user is following (e.g. followed users, followed datasets, followed groups).

+ id (string) – the id of the user

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=followee_count",
            "success": true,
            "result": 3
        }

## User followee count [/api/3/action/user_followee_count]

### Return the number of users that are followed by the given user. [GET]

Return the number of users that are followed by the given user.

+ id (string) – the id of the user

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=user_followee_count",
            "success": true,
            "result": 3
        }

## Dataset followee count [/api/3/action/dataset_followee_count]

### Return the number of datasets that are followed by the given user. [GET]

Return the number of datasets that are followed by the given user.

+ id (string) – the id of the user

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=dataset_followee_count",
            "success": true,
            "result": 1
        }

## Count groups for users [/api/3/action/group_followee_count]

### Get the number of groups that are followed by a user [GET]

Return the number of groups that are followed by the given user.

+ id (string) – the id of the user

+ Response 200 (application/json)

        {
            "help": "http://localhost:5000/api/3/action/help_show?name=group_followee_count", 
            "result": 2, 
            "success": true
        }


## Objects that a user follows [/api/3/action/followee_list]

### Get a list of objects that a user follows [GET]

Return the list of objects that are followed by the given user.

Returns all objects, of any type, that the given user is following (e.g. followed users, followed datasets, followed groups). This is a list of dictionaries, each with keys 'type' (e.g. 'user', 'dataset' or 'group'), 'display_name' (e.g. a user’s display name, or a package’s title) and 'dict' (e.g. a dict representing the followed user, package or group, the same as the dict that would be returned by user_show, package_show or group_show)

+ id (string) – the id of the user
+ q (string) – a query string to limit results by, only objects whose display name begins with the given string (case-insensitive) wil be returned (optional)

+ Response 200 (application/json)

        {
            "help": "http://localhost:5000/api/3/action/help_show?name=followee_list", 
            "result": [
                {
                    "dict": {
                        "approval_status": "approved", 
                        "created": "2015-09-08T14:12:21.580163", 
                        "description": "Description of the group", 
                        "display_name": "A dataset group", 
                        "extras": [], 
                        "groups": [], 
                        "id": "5fa0c44e-8ce5-4a4a-b1b4-c62f14388862", 
                        "image_display_url": "http://localhost:5000/uploads/group/2015-09-08-141221.528384image.gif", 
                        "image_url": "2015-09-08-141221.528384image.gif", 
                        "is_organization": false, 
                        "name": "a-dataset-group", 
                        "package_count": 0, 
                        "packages": [], 
                        "revision_id": "a0027dfe-bf36-45aa-b858-c5af5935c418", 
                        "state": "active", 
                        "tags": [], 
                        "title": "A dataset group", 
                        "type": "group", 
                        "users": [
                            {
                                "about": null, 
                                "activity_streams_email_notifications": false, 
                                "capacity": "admin", 
                                "created": "2014-10-14T09:53:31.608505", 
                                "display_name": "Person McFamily", 
                                "email_hash": "ecbd2a02951eefa421f34a57ec1c6e3b", 
                                "fullname": "Person McFamily", 
                                "id": "ea7a7ce9-f004-4646-b59b-f81aa5b4518e", 
                                "name": "person", 
                                "number_created_packages": 12, 
                                "number_of_edits": 106, 
                                "openid": null, 
                                "state": "active", 
                                "sysadmin": true
                            }
                        ]
                    }, 
                    "display_name": "A dataset group", 
                    "type": "group"
                }, 
                {
                    "dict": {
                        "about": null, 
                        "activity_streams_email_notifications": false, 
                        "created": "2015-03-26T11:37:10.801481", 
                        "display_name": "Ad Ministrator", 
                        "email_hash": "7220514c7b119071e1ff4438d9f5299c", 
                        "fullname": "Ad Ministrator", 
                        "id": "55d82268-66ca-4744-9f86-bc0cf8b66c58", 
                        "name": "iamadmin", 
                        "number_created_packages": 0, 
                        "number_of_edits": 1, 
                        "openid": null, 
                        "state": "active", 
                        "sysadmin": false
                    }, 
                    "display_name": "Ad Ministrator", 
                    "type": "user"
                }, 
                {
                    "dict": {
                        "approval_status": "approved", 
                        "created": "2015-09-08T12:21:40.893547", 
                        "description": "Description of organisation", 
                        "display_name": "Public body", 
                        "extras": [], 
                        "groups": [], 
                        "id": "a7acf343-0623-4be9-beb4-d24cbf3482f9", 
                        "image_display_url": "http://localhost:5000/uploads/group/2015-09-08-122140.699178pblcbdy.gif", 
                        "image_url": "2015-09-08-122140.699178pblcbdy.gif", 
                        "is_organization": true, 
                        "name": "public-body", 
                        "package_count": 0, 
                        "packages": [], 
                        "revision_id": "0bc898a9-dcfa-4037-b786-99d323735617", 
                        "state": "active", 
                        "tags": [], 
                        "title": "Public Body", 
                        "type": "organization", 
                        "users": [
                            {
                                "about": null, 
                                "activity_streams_email_notifications": false, 
                                "capacity": "admin", 
                                "created": "2014-10-14T09:53:31.608505", 
                                "display_name": "Person McFamily", 
                                "email_hash": "ecbd2a02951eefa421f34a57ec1c6e3b", 
                                "fullname": "Person McFamily", 
                                "id": "ea7a7ce9-f004-4646-b59b-f81aa5b4518e", 
                                "name": "person", 
                                "number_created_packages": 12, 
                                "number_of_edits": 106, 
                                "openid": null, 
                                "state": "active", 
                                "sysadmin": true
                            }
                        ]
                    }, 
                    "display_name": "Public Body", 
                    "type": "organization"
                }, 
                {
                    "dict": {
                        "author": null, 
                        "author_email": null, 
                        "creator_user_id": "ea7a7ce9-f004-4646-b59b-f81aa5b4518e", 
                        "extras": [], 
                        "groups": [], 
                        "id": "d9685ba1-989e-4724-8a11-37943c8d354c", 
                        "isopen": false, 
                        "license_id": null, 
                        "license_title": null, 
                        "maintainer": null, 
                        "maintainer_email": null, 
                        "metadata_created": "2015-02-26T13:16:05.694772", 
                        "metadata_modified": "2015-02-26T13:16:05.706062", 
                        "name": "testdataset", 
                        "notes": null, 
                        "num_resources": 0, 
                        "num_tags": 0, 
                        "organization": null, 
                        "owner_org": null, 
                        "private": false, 
                        "relationships_as_object": [], 
                        "relationships_as_subject": [], 
                        "resources": [], 
                        "revision_id": "5d43a9f0-7f5c-4fab-b608-6703d7057be5", 
                        "state": "active", 
                        "tags": [], 
                        "title": "testdataset", 
                        "type": "dataset", 
                        "url": null, 
                        "version": null
                    }, 
                    "display_name": "testdataset", 
                    "type": "dataset"
                }
            ], 
            "success": true
        }


## Users that a user follows [/api/3/action/user_followee_list]

### Get a list of users that a user follows [GET]

Return the list of users that are followed by the given user.

+ id (string) – the id of the user

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=user_followee_list", 
            "success": true,
            "result": [
                {
                    "about": null, 
                    "activity_streams_email_notifications": false, 
                    "created": "2015-03-26T11:37:10.801481", 
                    "display_name": "Ad Ministrator", 
                    "email_hash": "7220514c7b119071e1ff4438d9f5299c", 
                    "fullname": "Ad Ministrator", 
                    "id": "55d82268-66ca-4744-9f86-bc0cf8b66c58", 
                    "name": "iamadmin", 
                    "number_created_packages": 0, 
                    "number_of_edits": 1, 
                    "openid": null, 
                    "state": "active", 
                    "sysadmin": false
                }
            ]
        }


## Datasets that a user follows [/api/3/action/dataset_followee_list]

### Get a list of datasets that a user follows [GET]

Return the list of datasets that are followed by the given user.

+ id (string) – the id or name of the user

+ Response 200 (application/json)

        {
            "help": "http://localhost:5000/api/3/action/help_show?name=dataset_followee_list",
            "success": true,
            "result": [
                {
                    "author": null, 
                    "author_email": null, 
                    "creator_user_id": "ea7a7ce9-f004-4646-b59b-f81aa5b4518e", 
                    "extras": [], 
                    "groups": [], 
                    "id": "d9685ba1-989e-4724-8a11-37943c8d354c", 
                    "isopen": false, 
                    "license_id": null, 
                    "license_title": null, 
                    "maintainer": null, 
                    "maintainer_email": null, 
                    "metadata_created": "2015-02-26T13:16:05.694772", 
                    "metadata_modified": "2015-02-26T13:16:05.706062", 
                    "name": "testdataset", 
                    "notes": null, 
                    "num_resources": 0, 
                    "num_tags": 0, 
                    "organization": null, 
                    "owner_org": null, 
                    "private": false, 
                    "relationships_as_object": [], 
                    "relationships_as_subject": [], 
                    "resources": [], 
                    "revision_id": "5d43a9f0-7f5c-4fab-b608-6703d7057be5", 
                    "state": "active", 
                    "tags": [], 
                    "title": "testdataset", 
                    "type": "dataset", 
                    "url": null, 
                    "version": null
                }
            ]
        }


## Groups that a user follows [/api/3/action/group_followee_list]

### Get list of groups that a user follows [GET]

Return the list of groups that are followed by the given user.

+ id (string) – the id or name of the user

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=group_followee_list", 
            "success": true,
            "result": [
                {
                    "approval_status": "approved", 
                    "created": "2015-09-08T14:12:21.580163", 
                    "description": "Description of the group", 
                    "display_name": "A dataset group", 
                    "extras": [], 
                    "groups": [], 
                    "id": "5fa0c44e-8ce5-4a4a-b1b4-c62f14388862", 
                    "image_display_url": "http://demo.ckan.org/uploads/group/2015-09-08-141221.528384image.gif", 
                    "image_url": "2015-09-08-141221.528384image.gif", 
                    "is_organization": false, 
                    "name": "a-dataset-group", 
                    "package_count": 0, 
                    "packages": [], 
                    "revision_id": "a0027dfe-bf36-45aa-b858-c5af5935c418", 
                    "state": "active", 
                    "tags": [], 
                    "title": "A dataset group", 
                    "type": "group", 
                    "users": [
                        {
                            "about": null, 
                            "activity_streams_email_notifications": false, 
                            "capacity": "admin", 
                            "created": "2014-10-14T09:53:31.608505", 
                            "display_name": "Person McFamily", 
                            "email_hash": "ecbd2a02951eefa421f34a57ec1c6e3b", 
                            "fullname": "Person McFamily", 
                            "id": "ea7a7ce9-f004-4646-b59b-f81aa5b4518e", 
                            "name": "person", 
                            "number_created_packages": 12, 
                            "number_of_edits": 106, 
                            "openid": null, 
                            "state": "active", 
                            "sysadmin": true
                        }
                    ]
                }
            ]
        }


## Organisations that a user follows [/api/3/action/organization_followee_list]

### Get list of organisations that a user follows [GET]

Return the list of organizations that are followed by the given user.

+ id (string) – the id or name of the user

+ Response 200 (application/json)

        {
            "help": "http://localhost:5000/api/3/action/help_show?name=organization_followee_list", 
            "success": true
            "result": [
                {
                    "approval_status": "approved", 
                    "created": "2015-09-08T12:21:40.893547", 
                    "description": "Description of organisation", 
                    "display_name": "Public Body", 
                    "extras": [], 
                    "groups": [], 
                    "id": "a7acf343-0623-4be9-beb4-d24cbf3482f9", 
                    "image_display_url": "http://localhost:5000/uploads/group/2015-09-08-122140.699178pblcbdy.gif", 
                    "image_url": "2015-09-08-122140.699178pblcbdy.gif", 
                    "is_organization": true, 
                    "name": "public-body", 
                    "package_count": 0, 
                    "packages": [], 
                    "revision_id": "0bc898a9-dcfa-4037-b786-99d323735617", 
                    "state": "active", 
                    "tags": [], 
                    "title": "Public body", 
                    "type": "organization", 
                    "users": [
                        {
                            "about": null, 
                            "activity_streams_email_notifications": false, 
                            "capacity": "admin", 
                            "created": "2014-10-14T09:53:31.608505", 
                            "display_name": "Person McFamily", 
                            "email_hash": "ecbd2a02951eefa421f34a57ec1c6e3b", 
                            "fullname": "Person McFamily", 
                            "id": "ea7a7ce9-f004-4646-b59b-f81aa5b4518e", 
                            "name": "person", 
                            "number_created_packages": 12, 
                            "number_of_edits": 104, 
                            "openid": null, 
                            "state": "active", 
                            "sysadmin": true
                        }
                    ]
                }
            ]
        }


## Activity list [/api/3/action/dashboard_activity_list]

### Get the activities of the user's dashboard [GET]

Return the authorized user’s dashboard activity stream.

Unlike the activity dictionaries returned by other *_activity_list actions, these activity dictionaries have an extra boolean value with key is_new that tells you whether the activity happened since the user last viewed her dashboard ('is_new': True) or not ('is_new': False).

The user’s own activities are always marked 'is_new': False.

+ offset (int) – where to start getting activity items from (optional, default: 0)
+ limit – the maximum number of activities to return (optional, default: 31, the default value is configurable via the ckan.activity_list_limit setting)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=dashboard_activity_list", 
            "success": true,
            "result": [
                {
                    "activity_type": "new organization", 
                    "data": {
                        "group": {
                            "approval_status": "approved", 
                            "created": "2015-09-08T12:21:40.893547", 
                            "description": "Description of the new organisation", 
                            "id": "a7acf343-0623-4be9-beb4-d24cbf3482f9", 
                            "image_url": "2015-09-08-122140.699178coffeefail.gif", 
                            "is_organization": true, 
                            "name": "public-body", 
                            "revision_id": "0bc898a9-dcfa-4037-b786-99d323735617", 
                            "state": "active", 
                            "title": "Public Body", 
                            "type": "organization"
                        }
                    }, 
                    "id": "74448a33-a3bf-47db-a886-baf791f98eb7", 
                    "is_new": false, 
                    "object_id": "a7acf343-0623-4be9-beb4-d24cbf3482f9", 
                    "revision_id": "0bc898a9-dcfa-4037-b786-99d323735617", 
                    "timestamp": "2015-09-08T12:21:41.121911", 
                    "user_id": "ea7a7ce9-f004-4646-b59b-f81aa5b4518e"
                }
            ]
        }


## Activity list as HTML [/api/3/action/dashboard_activity_list_html]

### Get the activities of the user's dashboard as HTML [GET]

Return the authorized user’s dashboard activity stream as HTML.

The activity stream is rendered as a snippet of HTML meant to be included in an HTML page, i.e. it doesn’t have any HTML header or footer.

+ id (string) – the id or name of the user
+ offset (int) – where to start getting activity items from (optional, default: 0)
+ limit (int) – the maximum number of activities to return (optional, default: 31, the default value is configurable via the ckan.activity_list_limit setting)

+ Response 200 (application/json)

        {
            "help": "http://localhost:5000/api/3/action/help_show?name=dashboard_activity_list_html", 
            "success": true
            "result": "\n\n\n\n  \n    <ul class=\"activity\" data-module=\"activity-stream\" data-module-more=\"False\" data-module-context=\"user\" data-module-id=\"\" data-module-offset=\"0\">\n      \n        \n        \n          \n            \n<!-- Snippet snippets/activity_item.html start -->\n<li class=\"item new-organization\">\n  \n  <i class=\"icon icon-briefcase\"></i>\n  <p>\n    <span class=\"actor\"><img src=\"//gravatar.com/avatar/00000000000000000000000000000000?s=30&amp;d=identicon\"\n        class=\"gravatar\" width=\"30\" height=\"30\" /> <a href=\"/user/alice\">Tryggvi Björgvinsson</a></span> created the organization <a href=\"/organization/public-body\">public-body</a>\n    <span class=\"date\" title=\"September 8, 2015, 12:21\">41 minutes ago</span>\n  </p>\n</li>\n<!-- Snippet snippets/activity_item.html end -->\n\n          \n        \n        \n      \n    </ul>\n  \n", 
        }

## New activity count [/api/3/action/dashboard_new_activities_count]

### Get the number of new activities in the user's dashboard [GET]

Return the number of new activities in the authorized user’s dashboard activity stream.

Activities from the user herself are not counted by this function even though they appear in the dashboard (users don’t want to be notified about things they did themselves).

+ Response 200 (application/json)

        {
            "help": "http://localhost:5000/api/3/action/help_show?name=dashboard_new_activities_count", 
            "success": true,
            "result": 4
        }

## Member roles [/api/3/action/member_roles_list]

### Get list of member roles for organizations or groups [GET]

Return the possible roles for members of groups and organizations.

+ group_type (string) – the group type, either "group" or "organization" (optional, default "organization")

+ Response 200 (application/json)

        {
            "help": "http://localhost:5000/api/3/action/help_show?name=member_roles_list", 
            "success": true,
            "result": [
                {
                    "text": "Admin", 
                    "value": "admin"
                }, 
                {
                    "text": "Member", 
                    "value": "member"
                }
            ] 
        }

## API help text [/api/3/action/help_show]

### Get help text for an API call [GET]

Return the help string for a particular API action.

+ name (string) – Action function name (eg user_create, package_search)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=help_show", 
            "success": true,
            "result": "Return the help string for a particular API action..."
        }


## Single configuration option [/api/3/action/config_option_show]

### Get value for configuration option [GET]

Show the current value of a particular configuration option.

Only returns runtime-editable config options (the ones returned by config_option_list), which can be updated with the config_option_update action.
 
+ key (string) – The configuration option key

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=config_option_show", 
            "success": true,
            "result": "Test set of mine"
        }

## Configuration options list  [/api/3/action/config_option_list]

### Get list of configuration options [GET]

Return a list of runtime-editable config options keys that can be updated with config_option_update.

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=config_option_list", 
            "success": true
            "result": [
                "ckan.site_description", 
                "ckan.site_intro_text", 
                "ckan.site_custom_css", 
                "ckan.main_css", 
                "ckan.site_title", 
                "ckan.site_about", 
                "ckan.site_url", 
                "ckan.site_logo", 
                "ckan.homepage_style"
            ]
        }

# Group Creation functions

## Create a package [/api/3/action/package_create]

### Create a new dataset (package). [POST]

Create a new dataset (package).

You must be authorized to create new datasets. If you specify any groups for the new dataset, you must also be authorized to edit these groups.

Plugins may change the parameters of this function depending on the value of the type parameter, see the IDatasetForm plugin interface.

+ name (string) – the name of the new dataset, must be between 2 and 100 characters long and contain only lowercase alphanumeric characters, - and _, e.g. 'warandpeace'
+ title (string) – the title of the dataset (optional, default: same as name)
+ author (string) – the name of the dataset’s author (optional)
+ author_email (string) – the email address of the dataset’s author (optional)
+ maintainer (string) – the name of the dataset’s maintainer (optional)
+ maintainer_email (string) – the email address of the dataset’s maintainer (optional)
+ license_id (license id string) – the id of the dataset’s license, see license_list() for available values (optional)
+ notes (string) – a description of the dataset (optional)
+ url (string) – a URL for the dataset’s source (optional)
+ version (string, no longer than 100 characters) – (optional)
+ state (string) – the current state of the dataset, e.g. 'active' or 'deleted', only active datasets show up in search results and other lists of datasets, this parameter will be ignored if you are not authorized to change the state of the dataset (optional, default: 'active')
+ type (string) – the type of the dataset (optional), IDatasetForm plugins associate themselves with different dataset types and provide custom dataset handling behaviour for these types
+ resources (list of resource dictionaries) – the dataset’s resources, see resource_create() for the format of resource dictionaries (optional)h+ tags (list of tag dictionaries) – the dataset’s tags, see tag_create() for the format of tag dictionaries (optional)
+ tags (list of tag dictionaries) – the dataset’s tags, see tag_create() for the format of tag dictionaries (optional)
+ extras (list of dataset extra dictionaries) – the dataset’s extras (optional), extras are arbitrary (key: value) metadata items that can be added to datasets, each extra dictionary should have keys 'key' (a string), 'value' (a string)
+ relationships_as_object (list of relationship dictionaries) – see package_relationship_create() for the format of relationship dictionaries (optional)
+ relationships_as_subject (list of relationship dictionaries) – see package_relationship_create() for the format of relationship dictionaries (optional)
+ groups (list of dictionaries) – the groups to which the dataset belongs (optional), each group dictionary should have one or more of the following keys which identify an existing group: 'id' (the id of the group, string), or 'name' (the name of the group, string), to see which groups exist call group_list()
+ owner_org (string) – the id of the dataset’s owning organization, see organization_list() or organization_list_for_user() for available values (optional)


+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=package_create",
            "success": true,
            {
                "author": "author",
                "author_email": "author@ema.il",
                "creator_user_id": "3206bc4e-a870-4d4c-90d5-70c03bb343f6",
                "extras": [],
                "groups": [],
                "id": "c7c2267d-8e35-422d-83ac-05d382a1e726",
                "isopen": false,
                "license_id": null,
                "license_title": null,
                "maintainer": "maintainer",
                "maintainer_email": "maintainer@mail.com",
                "metadata_created": "2015-09-30T14:03:10.432280",
                "metadata_modified": "2015-09-30T14:03:10.638107",
                "name": "New dataset",
                "notes": "A dataset description",
                "num_resources": 0,
                "num_tags": 0,
                "organization": null,
                "owner_org": null,
                "private": false,
                "relationships_as_object": [],
                "relationships_as_subject": [],
                "resources": [],
                "revision_id": "a970db98-8867-4534-b23c-2b537a8de8f9",
                "state": "active",
                "tags": [],
                "title": "test",
                "type": "dataset",
                "url": null,
                "version": "version"
            }
        }


## Create a resource [/api/3/action/resource_create]

### Appends a new resource to a datasets list of resources. [POST]

+ package_id (string) – id of package that the resource should be added to.
+ url (string) – url of resource
+ revision_id (string) – (optional)
+ description (string) – (optional)
+ format (string) – (optional)
+ hash (string) – (optional)
+ name (string) – (optional)
+ resource_type (string) – (optional)
+ mimetype (string) – (optional)
+ mimetype_inner (string) – (optional)
+ webstore_url (string) – (optional)
+ cache_url (string) – (optional)
+ size (int) – (optional)
+ created (iso date string) – (optional)
+ last_modified (iso date string) – (optional)
+ cache_last_updated (iso date string) – (optional)
+ webstore_last_updated (iso date string) – (optional)
+ upload (multipart upload) – (optional)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=resource_create",
            "success": true,
            "result": {
                "cache_last_updated": null,
                "cache_url": null,
                "created": "2015-09-30T15:14:52.445023",
                "description": "My Resource",
                "format": "",
                "hash": "",
                "id": "c630210f-233f-4739-bf23-85be8373b5ee",
                "last_modified": null,
                "mimetype": null,
                "mimetype_inner": null,
                "name": null,
                "package_id": "644d0d3f-3570-4359-86de-3b12179c9f6d",
                "position": 0,
                "resource_type": null,
                "revision_id": "d638cb93-97ad-405b-a79d-bb70940e5792",
                "size": null,
                "state": "active",
                "url": "http://test.com",
                "url_type": null,
                "webstore_last_updated": null,
                "webstore_url": null
            }
        }


## Create a resource view [/api/3/action/resource_view_create]

### Creates a new resource view. [POST]

+ resource_id (string) – id of the resource
+ title (string) – the title of the view
+ description (string) – a description of the view (optional)
+ view_type (string) – type of view
+ config (JSON string) – options necessary to recreate a view state (optional)


+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=resource_view_create",
            "success": true,
            "result": {
                "description": "A description",
                "id": "7af23c9a-8e46-4e1f-ab76-ae400f6ab98e",
                "package_id": "644d0d3f-3570-4359-86de-3b12179c9f6d",
                "resource_id": "c630210f-233f-4739-bf23-85be8373b5ee",
                "title": "My resource view",
                "view_type": "text_view"
            }
        }


## Create package relationship  [/api/3/action/package_relationship_create]

### Create a relationship between two datasets (packages). [POST]

Create a relationship between two datasets (packages).

You must be authorized to edit both the subject and the object datasets.

+ subject (string) – the id or name of the dataset that is the subject of the relationship
+ object – the id or name of the dataset that is the object of the relationship
+ type (string) – the type of the relationship, one of 'depends_on', 'dependency_of', 'derives_from', 'has_derivation', 'links_to', 'linked_from', 'child_of' or 'parent_of'
+ comment (string) – a comment about the relationship (optional)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=package_relationship_create",
            "success": true,
            "result": {
                "comment": "A comment",
                "object": "another",
                "subject": "test_dataset",
                "type": "child_of
            }
        }

## Create a member [/api/3/action/member_create]

### Make an object (e.g. a user, dataset or group) a member of a group. [POST]

Make an object (e.g. a user, dataset or group) a member of a group.

If the object is already a member of the group then the capacity of the membership will be updated.

You must be authorized to edit the group.

+ id (string) – the id or name of the group to add the object to
+ object (string) – the id or name of the object to add
+ object_type (string) – the type of the object being added, e.g. 'package' or 'user'
+ capacity (string) – the capacity of the membership

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=member_create",
            "success": true,
            "result": {
                "capacity": "belong",
                "group_id": "af18fb7f-4ad3-429d-be64-83fb10209999",
                "id": "93be1edf-e4d2-4e59-b62f-cbefcd74f76a",
                "revision_id": "3611c419-3793-49fa-8ef7-26413bb55865",
                "state": "active",
                "table_id": "644d0d3f-3570-4359-86de-3b12179c9f6d",
                "table_name": "package"
            }
        }

## Create a group [/api/3/action/group_create]

### Create a new group. [POST]
Create a new group.

You must be authorized to create groups.

Plugins may change the parameters of this function depending on the value of the type parameter, see the IGroupForm plugin interface.

+ name (string) – the name of the group, a string between 2 and 100 characters long, containing only lowercase alphanumeric characters, - and _
+ id (string) – the id of the group (optional)
+ title (string) – the title of the group (optional)
+ description (string) – the description of the group (optional)
+ image_url (string) – the URL to an image to be displayed on the group’s page (optional)
+ type (string) – the type of the group (optional), IGroupForm plugins associate themselves with different group types and provide custom group handling behaviour for these types Cannot be ‘organization’
+ state (string) – the current state of the group, e.g. 'active' or 'deleted', only active groups show up in search results and other lists of groups, this parameter will be ignored if you are not authorized to change the state of the group (optional, default: 'active')
+ approval_status (string) – (optional)
+ extras (list of dataset extra dictionaries) – the group’s extras (optional), extras are arbitrary (key: value) metadata items that can be added to groups, each extra dictionary should have keys 'key' (a string), 'value' (a string), and optionally 'deleted'
+ packages (list of dictionaries) – the datasets (packages) that belong to the group, a list of dictionaries each with keys 'name' (string, the id or name of the dataset) and optionally 'title' (string, the title of the dataset)
+ groups (list of dictionaries) – the groups that belong to the group, a list of dictionaries each with key 'name' (string, the id or name of the group) and optionally 'capacity' (string, the capacity in which the group is a member of the group)
+ users (list of dictionaries) – the users that belong to the group, a list of dictionaries each with key 'name' (string, the id or name of the user) and optionally 'capacity' (string, the capacity in which the user is a member of the group)


+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=member_create",
            "success": true,
            "result": {
                "approval_status": "approved",
                "created": "2015-09-30T15:42:00.644667",
                "description": "",
                "display_name": "test",
                "extras": [],
                "groups": [],
                "id": "af18fb7f-4ad3-429d-be64-83fb10209999",
                "image_display_url": "",
                "image_url": "",
                "is_organization": false,
                "name": "test",
                "num_followers": 0,
                "package_count": 0,
                "revision_id": "c635e82d-b73a-4ded-9908-31e7bd7021a8",
                "state": "active",
                "tags": [],
                "title": "",
                "type": "group",
                "users": [
                {
                    "about": null,
                    "activity_streams_email_notifications": false,
                    "capacity": "admin",
                    "created": "2015-09-30T15:13:29.569025",
                    "display_name": "default",
                    "email_hash": "d41d8cd98f00b204e9800998ecf8427e",
                    "fullname": null,
                    "id": "4e572675-ed8e-4353-91ce-215c3fa44d85",
                    "name": "default",
                    "number_created_packages": 2,
                    "number_of_edits": 6,
                    "openid": null,
                    "state": "active",
                    "sysadmin": true
                }
                ]

            }
        }

## Crete an organization [/api/3/action/organization_create]

### Create a new organization. [POST]

Create a new organization.

You must be authorized to create organizations.

Plugins may change the parameters of this function depending on the value of the type parameter, see the IGroupForm plugin interface.

+ name (string) – the name of the organization, a string between 2 and 100 characters long, containing only lowercase alphanumeric characters, - and _
+ id (string) – the id of the organization (optional)
+ title (string) – the title of the organization (optional)
+ description (string) – the description of the organization (optional)
+ image_url (string) – the URL to an image to be displayed on the organization’s page (optional)
+ state (string) – the current state of the organization, e.g. 'active' or 'deleted', only active organizations show up in search results and other lists of organizations, this parameter will be ignored if you are not authorized to change the state of the organization (optional, default: 'active')
+ approval_status (string) – (optional)
+ extras (list of dataset extra dictionaries) – the organization’s extras (optional), extras are arbitrary (key: value) metadata items that can be added to organizations, each extra dictionary should have keys 'key' (a string), 'value' (a string), and optionally 'deleted'
+ packages (list of dictionaries) – the datasets (packages) that belong to the organization, a list of dictionaries each with keys 'name' (string, the id or name of the dataset) and optionally 'title' (string, the title of the dataset)
+ users (list of dictionaries) – the users that belong to the organization, a list of dictionaries each with key 'name' (string, the id or name of the user) and optionally 'capacity' (string, the capacity in which the user is a member of the organization)


+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=organization_create",
            "success": true,
            "result": {
                "approval_status": "approved",
                "created": "2015-09-30T15:47:42.377056",
                "description": "",
                "display_name": "new_organization",
                "extras": [],
                "groups": [],
                "id": "086ffbdc-fa46-4cb5-afe4-7d598d69687b",
                "image_display_url": "",
                "image_url": "",
                "is_organization": true,
                "name": "new_organization",
                "num_followers": 0,
                "package_count": 0,
                "revision_id": "4fbea46b-776c-4d4b-870b-9dfb0fa462a5",
                "state": "active",
                "tags": [],
                "title": "",
                "type": "organization",
                "users": [
                {
                    "about": null,
                    "activity_streams_email_notifications": false,
                    "capacity": "admin",
                    "created": "2015-09-30T15:13:29.569025",
                    "display_name": "user",
                    "email_hash": "d41d8cd98f00b204e9800998ecf8427e",
                    "fullname": null,
                    "id": "4e572675-ed8e-4353-91ce-215c3fa44d85",
                    "name": "default",
                    "number_created_packages": 2,
                    "number_of_edits": 9,
                    "openid": null,
                    "state": "active",
                    "sysadmin": false
                }
                ]
            }
        }


## Create user  [/api/3/action/user_create]

### Create a new user. [POST]

Create a new user.

You must be authorized to create users.

+ name (string) – the name of the new user, a string between 2 and 100 characters in length, containing only lowercase alphanumeric characters, - and _
+ email (string) – the email address for the new user
+ password (string) – the password of the new user, a string of at least 4 characters
+ id (string) – the id of the new user (optional)
+ fullname (string) – the full name of the new user (optional)
+ about (string) – a description of the new user (optional)
+ openid (string) – (optional)


+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=user_create",
            "success": true,
            "result": {
                "about": null,
                "activity_streams_email_notifications": false,
                "apikey": "7cfb26d3-6e31-4068-a178-fb0c1df775c6",
                "created": "2015-09-30T16:06:32.927405",
                "display_name": "name",
                "email": "email@email.net",
                "email_hash": "18ae5542886131d75c1dbdd117bab38a",
                "fullname": null,
                "id": "89c185f0-e005-485b-9cf6-f9364475c536",
                "name": "name",
                "number_created_packages": 0,
                "number_of_edits": 0,
                "openid": null,
                "state": "active",
                "sysadmin": false
            }
        }

## Invite a user [/api/3/action/user_invite]

### Invite a new user. [POST]

Invite a new user.

You must be authorized to create group members.

+ email (string) – the email of the user to be invited to the group
+ group_id (string) – the id or name of the group
+ role (string) – role of the user in the group. One of member, editor, or admin

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=user_invite",
            "success": true,
            "result": {
                "about": null,
                "activity_streams_email_notifications": false,
                "apikey": "7cfb26d3-6e31-4068-a178-fb0c1df775c6",
                "created": "2015-09-30T16:06:32.927405",
                "display_name": "name",
                "email": "email@email.net",
                "email_hash": "18ae5542886131d75c1dbdd117bab38a",
                "fullname": null,
                "id": "89c185f0-e005-485b-9cf6-f9364475c536",
                "name": "name",
                "number_created_packages": 0,
                "number_of_edits": 0,
                "openid": null,
                "state": "active",
                "sysadmin": false
            }
        }

## Vocabulary create [/api/3/action/vocabulary_create]

### Create a new tag vocabulary. [POST]
Create a new tag vocabulary.

You must be a sysadmin to create vocabularies.

+ name (string) – the name of the new vocabulary, e.g. 'Genre'
+ tags (list of tag dictionaries) – the new tags to add to the new vocabulary, for the format of tag dictionaries see tag_create()

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=vocabulary_create",
            "success": true,
            "result": {
                "id": "d77a56bf-e892-4731-ab92-e8032813965c",
                "name": "genre",
                "tags": []
            }
        }

## Activity create [/api/3/action/activity_create]

### Create a new activity stream activity. [POST]

Create a new activity stream activity.

You must be a sysadmin to create new activities.

+ user_id (string) – the name or id of the user who carried out the activity, e.g. 'seanh'
+ object_id – the name or id of the object of the activity, e.g. 'my_dataset'
+ activity_type (string) – the type of the activity, this must be an activity type that CKAN knows how to render, e.g. 'new package', 'changed user', 'deleted group' etc.
+ data (dictionary) – any additional data about the activity

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=activity_create",
            "success": true,
            "result": {
                "activity_type": "new package",
                "data": {},
                "id": "8d85c4a1-8d38-4506-ab45-dc2a12d1e5a4",
                "object_id": "644d0d3f-3570-4359-86de-3b12179c9f6d",
                "revision_id": null,
                "timestamp": "2015-09-30T16:23:45.264706",
                "user_id": "89c185f0-e005-485b-9cf6-f9364475c536"
            }
        }

## Create a vocabulary tag [/api/3/action/tag_create]

### Create a new vocabulary tag. [POST]

Create a new vocabulary tag.

You must be a sysadmin to create vocabulary tags.

You can only use this function to create tags that belong to a vocabulary, not to create free tags. (To create a new free tag simply add the tag to a package, e.g. using the package_update() function.)

+ name (string) – the name for the new tag, a string between 2 and 100 characters long containing only alphanumeric characters and -, _ and ., e.g. 'Jazz'
+ vocabulary_id (string) – the id of the vocabulary that the new tag should be added to, e.g. the id of vocabulary 'Genre'

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=tag_create",
            "success": true,
            "result": {
                "display_name": "jazz",
                "id": "1ede4083-c9a8-4d9b-abc6-e3a626f1a6c8",
                "name": "jazz",
                "packages": [],
                "vocabulary_id": "d77a56bf-e892-4731-ab92-e8032813965c"
            }
        }

## Follow user [/api/3/action/follow_user]

### Start following another user. [POST]
Start following another user.

You must provide your API key in the Authorization header.

+ id (string) – the id or name of the user to follow, e.g. 'joeuser'

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=follow_user",
            "success": true,
            "result": {
                "datetime": "2015-09-30T12:29:35.316562",
                "follower_id": "4e572675-ed8e-4353-91ce-215c3fa44d85",
                "object_id": "89c185f0-e005-485b-9cf6-f9364475c536"
            }
        }


## Follow dataset  [/api/3/action/follow_dataset]

Start following a dataset.

You must provide your API key in the Authorization header.
+ id (string) – the id or name of the dataset to follow, e.g. 'warandpeace'

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=follow_dataset",
            "success": true,
            "result": {
                "datetime": "2015-09-30T12:30:34.391110",
                "follower_id": "4e572675-ed8e-4353-91ce-215c3fa44d85",
                "object_id": "644d0d3f-3570-4359-86de-3b12179c9f6d"
            }
        }


## Create a group member [/api/3/action/group_member_create]

### Make a user a member of a group. [POST]

Make a user a member of a group.

You must be authorized to edit the group.

+ id (string) – the id or name of the group
+ username (string) – name or id of the user to be made member of the group
+ role (string) – role of the user in the group. One of member, editor, or admin


+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=group_member_create",
            "success": true,
            "result": {
                "capacity": "member",
                "group_id": "af18fb7f-4ad3-429d-be64-83fb10209999",
                "id": "e04e12a4-0f09-471a-a4f3-79c8b9531702",
                "revision_id": "86efe388-8765-4c3d-a895-5d2b7cda9dca",
                "state": "active",
                "table_id": "89c185f0-e005-485b-9cf6-f9364475c536",
                "table_name": "user"
            }
        }

## Organization member create  [/api/3/action/organization_member_create]

### Make a user a member of an organization. [POST]
Make a user a member of an organization.

You must be authorized to edit the organization.

+ id (string) – the id or name of the organization
+ username (string) – name or id of the user to be made member of the organization
+ role (string) – role of the user in the organization. One of member, editor, or admin

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=organization_member_create",
            "success": true,
            "result": {
                "capacity": "member",
                "group_id": "af18fb7f-4ad3-429d-be64-83fb10209999",
                "id": "e04e12a4-0f09-471a-a4f3-79c8b9531702",
                "revision_id": "86efe388-8765-4c3d-a895-5d2b7cda9dca",
                "state": "active",
                "table_id": "89c185f0-e005-485b-9cf6-f9364475c536",
                "table_name": "user"
            }
        }

## Follow a group  [/api/3/action/follow_group]

### Start following a group. [POST]
Start following a group.

You must provide your API key in the Authorization header.
+ id (string) – the id or name of the group to follow, e.g. 'roger'

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=package_create",
            "success": true,
            "result": {
                "datetime": "2015-09-30T14:41:49.346662",
                "follower_id": "4e572675-ed8e-4353-91ce-215c3fa44d85",
                "object_id": "af18fb7f-4ad3-429d-be64-83fb10209999"
            }
        }


# Group Update functions

API functions for updating existing data in CKAN.

## Update a resource [/api/3/action/resource_update]

### Update a specific resource [POST]

To update a resource you must be authorized to update the dataset that the resource belongs to.

+ id (string) – the id of the resource to update
+ url (string) – URL pointing to the resource. Must be provided, even if it should not be updated (should then include the same value)
+ name (string) – Name of the resource. Optional
+ description (string) – Description of the resource. Optional
+ mimetype (string) – Mimetype of resource. Optional
+ upload (multipart upload) - Resource file to upload (instead of a url). Optional

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=resource_update", 
            "result": {
                "cache_last_updated": null, 
                "cache_url": null, 
                "created": "2015-09-18T17:18:03.488178", 
                "description": "",   
                "format": "CSV", 
                "hash": "", 
                "id": "78a8e205-a9be-4fcc-8f95-5f11708ae09f", 
                "last_modified": "2015-09-18T17:18:03.488178", 
                "mimetype": "text/csv", 
                "mimetype_inner": null, 
                "name": "boost-armenia", 
                "package_id": "144ad431-9e23-422e-be20-4b1e8c289d00", 
                "position": 0, 
                "resource_type": null, 
                "revision_id": "5e9fd56b-08e5-49c8-b52e-95944fe7f6ae", 
                "size": "9324912", 
                "state": "active", 
                "url": "https://example.com/resource.csv", 
                "url_type": null, 
                "webstore_last_updated": null, 
                "webstore_url": null
            }, 
            "success": true
        }


## Update resource view [/api/3/action/resource_view_update]

### Update a specific resource view [POST]

To update a resource_view you must be authorized to update the resource that the resource_view belongs to.

+ id (string) – the id of the resource_view to update
+ resource_id (string) – id of the resource
+ title (string) – The title of the view
+ description (string) – A description of the view. Optional
+ view_type (string) – The type of a resource view
+ config (JSON string) – Options necessary to recreate a view state. Optional

+ Response 200 (application/json)

        {
            "help": "http://localhost:5000/api/3/action/help_show?name=resource_view_update", 
            "result": {
                "description": "This is a resource view description", 
                "id": "27d772d5-96f6-4c7d-b0a9-f3f954745505", 
                "package_id": "144ad431-9e23-422e-be20-4b1e8c289d00", 
                "resource_id": "78a8e205-a9be-4fcc-8f95-5f11708ae09f", 
                "title": "Resource view title", 
                "view_type": "recline_view"
            }, 
            "success": true
        }


## Reorder resource views [/api/3/action/resource_view_reorder]

### Reorder resource views for a specific resource [POST]

Reorder resource views.

+ id (string) – the id of the resource
+ order (list of strings) – the list of id of the resource to update the order of the views

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=resource_view_reorder",
            "result": {
                "id": "78a8e205-a9be-4fcc-8f95-5f11708ae09f",
                "order": [
                    "27d772d5-96f6-4c7d-b0a9-f3f954745505",
                    "a475a27a-f5ab-4470-891a-957484268dc6"
                ]
            },
            "success": true
        }


## Update a package [/api/3/action/package_update]

### Update a specific dataset [POST]

Update a dataset (package). You must be authorized to edit the dataset and the groups that it belongs to. It is recommended to /api/3/action/package_show, make the desired changes to the result, and then call this function with the updated dictionary.

Plugins may change the parameters of this function depending on the value of the dataset’s type attribute, see the IDatasetForm plugin interface.

+ id (string) – the name or id of the dataset to update
+ title (string) – the title of the dataset. Optional
+ author (string) – the name of the dataset’s author. Optional
+ author_email (string) – the email address of the dataset’s author. Optional
+ maintainer (string) – the name of the dataset’s maintainer. Optional
+ maintainer_email (string) – the email address of the dataset’s maintainer. Optional
+ license_id (license id string) – the id of the dataset’s license, see /api/3/action/license_list for available values. Optional
+ notes (string) – a description of the dataset. Optional
+ url (string) – a URL for the dataset’s source. Optional
+ version (string, no longer than 100 characters) – Optional
+ state (string) – the current state of the dataset, e.g. 'active' or 'deleted', only active datasets show up in search results and other lists of datasets, this parameter will be ignored if you are not authorized to change the state of the dataset. Optional, default: 'active'
+ type (string) – the type of the dataset, IDatasetForm plugins associate themselves with different dataset types and provide custom dataset handling behaviour for these types. Optional
+ resources (list of resource dictionaries) – the dataset’s resources, see /api/3/action/resource_create for the format of resource dictionaries. Note: The complete resource list must be provided or else this update will be overwritten.
+ tags (list of tag dictionaries) – the dataset’s tags, see /api/3/action/tag_create for the format of tag dictionaries. Optional
+ extras (list of dataset extra dictionaries) – the dataset’s extras, extras are arbitrary (key: value) metadata items that can be added to datasets, each extra dictionary should have keys 'key' (a string), 'value' (a string). Optional
+ groups (list of dictionaries) – the groups to which the dataset belongs, each group dictionary should have one or more of the following keys which identify an existing group: 'id' (the id of the group, string), or 'name' (the name of the group, string), to see which groups exist call /api/3/action/group_list. Optional
+ owner_org (string) – the id of the dataset’s owning organization, see /api/3/action/organization_list or /api/3/action/organization_list_for_user for available values. Optional

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=package_update", 
            "result": {
                "author": "", 
                "author_email": "", 
                "creator_user_id": "ea7a7ce9-f004-4646-b59b-f81aa5b4518e",  
                "extras": [], 
                "groups": [], 
                "id": "4e77badd-cd28-4825-97f1-9e2dffdbb002", 
                "isopen": true, 
                "license_id": "cc-by-sa", 
                "license_title": "Creative Commons Attribution Share-Alike", 
                "license_url": "http://www.opendefinition.org/licenses/cc-by-sa", 
                "maintainer": "", 
                "maintainer_email": "", 
                "metadata_created": "2015-09-30T11:01:07.264156", 
                "metadata_modified": "2015-09-30T11:02:54.254457", 
                "name": "a-dataset", 
                "notes": "A dataset description", 
                "num_resources": 0, 
                "num_tags": 0, 
                "organization": {
                    "approval_status": "approved", 
                    "created": "2015-09-08T20:20:15.147539", 
                    "description": "Description for this other organisation", 
                    "id": "a3c4b4b8-961d-45f4-8981-728653f36445", 
                    "image_url": "2015-09-08-202015.059790image.gif", 
                    "is_organization": true, 
                    "name": "another-organization", 
                    "revision_id": "a6b9d4be-d7da-48d6-8c14-f170f03e899e", 
                    "state": "active", 
                    "title": "Another organization", 
                    "type": "organization"
                }, 
                "owner_org": "a3c4b4b8-961d-45f4-8981-728653f36445", 
                "private": false, 
                "relationships_as_object": [], 
                "relationships_as_subject": [], 
                "resources": [], 
                "revision_id": "d937d0d2-ac8a-419a-b987-66c74a1eec5d", 
                "state": "active", 
                "tags": [], 
                "title": "The dataset title", 
                "type": "dataset", 
                "url": "", 
                "version": ""
            }, 
            "success": true
        }

## Reorder package resources [/api/3/action/package_resource_reorder]

### Reorder resource in a specific package [POST]

Reorder resources against datasets. If only partial resource ids are supplied then these are assumed to be first and the other resources will stay in their original order.

+ id (string) – the id or name of the package to update
+ order (list of strings) – a list of resource ids in the order needed

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=package_resource_reorder",
            "success": true,
            "result": {
                "id": "a-dataset",
                "order": [
                    "248ed31b-bab5-4a0e-bb3e-9c03c2ffebf8",
                    "07889ab7-ae3b-44a3-8d68-c2b83b449541"
                ]
            }
        }

## Update group [/api/3/action/group_update]

### Update a specific group [POST]

Update a group. You must be authorized to edit the group.

Plugins may change the parameters of this function depending on the value of the group’s type attribute, see the IGroupForm plugin interface.

+ id (string) – the name or id of the group to update
+ title (string) – the title of the group. Optional
+ description (string) – the description of the group. Optional
+ image_url (string) – the URL to an image to be displayed on the group’s page. Optional
+ type (string) – the type of the group, IGroupForm plugins associate themselves with different group types and provide custom group handling behaviour for these types Cannot be ‘organization’. Optional
+ state (string) – the current state of the group, e.g. 'active' or 'deleted', only active groups show up in search results and other lists of groups, this parameter will be ignored if you are not authorized to change the state of the group. Optional, default: 'active'
+ extras (list of dataset extra dictionaries) – the group’s extras, extras are arbitrary (key: value) metadata items that can be added to groups, each extra dictionary should have keys 'key' (a string), 'value' (a string), and optionally 'deleted'. Optional
+ packages (list of dictionaries) – the datasets (packages) that belong to the group, a list of dictionaries each with keys 'name' (string, the id or name of the dataset) and optionally 'title' (string, the title of the dataset)
+ groups (list of dictionaries) – the groups that belong to the group, a list of dictionaries each with key 'name' (string, the id or name of the group) and optionally 'capacity' (string, the capacity in which the group is a member of the group)
+ users (list of dictionaries) – the users that belong to the group, a list of dictionaries each with key 'name' (string, the id or name of the user) and optionally 'capacity' (string, the capacity in which the user is a member of the group)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=group_update",
            "result": {
                "approval_status": "approved", 
                "created": "2015-09-08T14:12:21.580163", 
                "description": "Description of the group", 
                "display_name": "The group title", 
                "extras": [], 
                "groups": [], 
                "id": "5fa0c44e-8ce5-4a4a-b1b4-c62f14388862", 
                "image_display_url": "http://127.0.0.1:5000/uploads/group/2015-09-08-141221.528384image.gif", 
                "image_url": "2015-09-08-141221.528384image.gif", 
                "is_organization": false, 
                "name": "a-dataset-group", 
                "package_count": 0, 
                "packages": [], 
                "revision_id": "11cd8aa9-bfe6-4ab8-aeed-b8dfe4a68850", 
                "state": "active", 
                "tags": [], 
                "title": "The group title", 
                "type": "group", 
                "users": []
            }, 
            "success": true
        }


## Update organisation [/api/3/action/organization_update]

### Update a specific organization [POST]

Update a organization. You must be authorized to edit the organization.

+ id (string) – the name or id of the organization to update
+ title (string) – the title of the organization. Optional
+ description (string) – the description of the organization. Optional
+ image_url (string) – the URL to an image to be displayed on the organization’s page. Optional
+ state (string) – the current state of the organization, e.g. 'active' or 'deleted', only active organizations show up in search results and other lists of organizations, this parameter will be ignored if you are not authorized to change the state of the organization. Optional, default: 'active'
+ extras (list of dataset extra dictionaries) – the organization’s extras, extras are arbitrary (key: value) metadata items that can be added to organizations, each extra dictionary should have keys 'key' (a string), 'value' (a string), and optionally 'deleted'. Optional
+ users (list of dictionaries) – the users that belong to the organization, a list of dictionaries each with key 'name' (string, the id or name of the user) and optionally 'capacity' (string, the capacity in which the user is a member of the organization)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=organization_update", 
            "result": {
                "approval_status": "approved", 
                "created": "2015-09-08T20:20:15.147539", 
                "description": "Description for this other organisation", 
                "display_name": "A different organization", 
                "extras": [], 
                "groups": [], 
                "id": "a3c4b4b8-961d-45f4-8981-728653f36445", 
                "image_display_url": "http://127.0.0.1:5000/uploads/group/2015-09-08-202015.059790image.gif", 
                "image_url": "2015-09-08-202015.059790image.gif", 
                "is_organization": true, 
                "name": "another-organization", 
                "package_count": 1, 
                "packages": [
                    {
                        "author": "", 
                        "author_email": "", 
                        "creator_user_id": "ea7a7ce9-f004-4646-b59b-f81aa5b4518e", 
                        "extras": [], 
                        "groups": [], 
                        "id": "4e77badd-cd28-4825-97f1-9e2dffdbb002", 
                        "isopen": true, 
                        "license_id": "cc-by-sa", 
                        "license_title": "Creative Commons Attribution Share-Alike", 
                        "license_url": "http://www.opendefinition.org/licenses/cc-by-sa", 
                        "maintainer": "", 
                        "maintainer_email": "", 
                        "metadata_created": "2015-09-30T11:01:07.264156", 
                        "metadata_modified": "2015-09-30T11:39:58.823983", 
                        "name": "a-dataset", 
                        "notes": "A dataset description", 
                        "num_resources": 2, 
                        "num_tags": 0, 
                        "organization": {
                            "approval_status": "approved", 
                            "created": "2015-09-08T20:20:15.147539", 
                            "description": "Description for this other organisation", 
                            "id": "a3c4b4b8-961d-45f4-8981-728653f36445", 
                            "image_url": "2015-09-08-202015.059790image.gif", 
                            "is_organization": true, 
                            "name": "another-organization", 
                            "revision_id": "a6b9d4be-d7da-48d6-8c14-f170f03e899e", 
                            "state": "active", 
                            "title": "Another organization", 
                            "type": "organization"
                        }, 
                        "owner_org": "a3c4b4b8-961d-45f4-8981-728653f36445", 
                        "private": false, 
                        "resources": [
                            {
                                "cache_last_updated": null, 
                                "cache_url": null, 
                                "created": "2015-09-30T11:35:00.174546", 
                                "description": "Description of other resource", 
                                "format": "CSV", 
                                "hash": "", 
                                "id": "248ed31b-bab5-4a0e-bb3e-9c03c2ffebf8", 
                                "last_modified": null, 
                                "mimetype": null, 
                                "mimetype_inner": null, 
                                "name": "Another resource", 
                                "package_id": "4e77badd-cd28-4825-97f1-9e2dffdbb002", 
                                "position": 0, 
                                "resource_type": null, 
                                "revision_id": "db723a9d-969e-43fd-8a50-828ac99dddcb", 
                                "size": null, 
                                "state": "active", 
                                "url": "http://example.com/another-dataset.csv", 
                                "url_type": null, 
                                "webstore_last_updated": null, 
                                "webstore_url": null
                            }, 
                            {
                                "cache_last_updated": null, 
                                "cache_url": null, 
                                "created": "2015-09-30T11:34:26.364593", 
                                "description": "A resource description", 
                                "format": "CSV", 
                                "hash": "", 
                                "id": "07889ab7-ae3b-44a3-8d68-c2b83b449541", 
                                "last_modified": null, 
                                "mimetype": null, 
                                "mimetype_inner": null, 
                                "name": "A resource", 
                                "package_id": "4e77badd-cd28-4825-97f1-9e2dffdbb002", 
                                "position": 1, 
                                "resource_type": null, 
                                "revision_id": "db723a9d-969e-43fd-8a50-828ac99dddcb", 
                                "size": null, 
                                "state": "active", 
                                "url": "http://example.com/dataset.csv", 
                                "url_type": null, 
                                "webstore_last_updated": null, 
                                "webstore_url": null
                            }
                        ], 
                        "revision_id": "d937d0d2-ac8a-419a-b987-66c74a1eec5d", 
                        "state": "active", 
                        "tags": [], 
                        "title": "The dataset title", 
                        "type": "dataset", 
                        "url": "", 
                        "version": ""
                    }
                ], 
                "revision_id": "8c96920d-5676-4ba4-b465-2e48fb17efa3", 
                "state": "active", 
                "tags": [], 
                "title": "A different organization", 
                "type": "organization", 
                "users": []
            }, 
            "success": true
        }

## Update user [/api/3/action/user_update]

### Update a user's account [POST]

Update a user account. Normal users can only update their own user accounts. Sysadmins can update any user account.

+ id (string) – The name or id of the user to update
+ email (string) – The email address for the new user
+ password (string) – The password of the new user, a string of at least 4 characters
+ fullname (string) – The full name of the new user. Optional
+ about (string) – A description of the new user. Optional
+ openid (string) – The OpenID for the user. Optional

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=user_update", 
            "result": {
                "about": null, 
                "activity_streams_email_notifications": false, 
                "apikey": "2d8ee7e1-f7e1-4dce-af55-17543ff56bfd", 
                "created": "2014-10-14T09:52:54.254003", 
                "display_name": "Tryggvi Björgvinsson", 
                "email": "me@email.com", 
                "email_hash": "8f9dc04e6abdcc9fea53e81945c7294b", 
                "fullname": "Tryggvi Björgvinsson", 
                "id": "922344d1-4362-4b42-9c38-5edcfaf3f1cc", 
                "name": "trickvi", 
                "number_created_packages": 0, 
                "number_of_edits": 0, 
                "openid": null, 
                "state": "active", 
                "sysadmin": true
            }, 
            "success": true
        }


## Regenerate API key [/api/3/action/user_generate_apikey]

### Generate a new API key for a user [POST]

Cycle a user’s API key

+ id (string) – the name or id of the user whose key needs to be updated

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=user_generate_apikey", 
            "result": {
                "about": null, 
                "activity_streams_email_notifications": false, 
                "apikey": "fd74af5f-1c71-420b-a397-825c0e63e253", 
                "created": "2014-10-14T09:52:54.254003", 
                "display_name": "Tryggvi Björgvinsson", 
                "email": "me@email.com", 
                "email_hash": "8f9dc04e6abdcc9fea53e81945c7294b", 
                "fullname": "Tryggvi Björgvinsson", 
                "id": "922344d1-4362-4b42-9c38-5edcfaf3f1cc", 
                        "name": "trickvi", 
                "number_created_packages": 0, 
                "number_of_edits": 0, 
                "openid": null, 
                "state": "active", 
                "sysadmin": true
            }, 
            "success": true
        }


## Mark activities as old [/api/3/action/dashboard_mark_activities_old]

### Mark user's activities as old [POST]

Mark all the authorized user’s new dashboard activities as old. This will reset /api/3/action/dashboard_new_activities_count to 0.

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=dashboard_mark_activities_old",
            "success": true,
            "result": null
        }


## Make datasets private [/api/3/action/bulk_update_private]

### Set a list of datasets to visibility status private [POST]

Make a list of datasets private

+ datasets (list of strings) – list of ids of the datasets to update
+ org_id (int) – id of the owning organization

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=bulk_update_private",
            "success": true,
            "result": null
        }


## Make datasets public [/api/3/action/bulk_update_public]

### Set a list of datasets to visibility status public [POST]

Make a list of datasets public

+ datasets (list of strings) – list of ids of the datasets to update
+ org_id (int) – id of the owning organization

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=bulk_update_public",
            "success": true,
            "result": null
        }

## Delete a list of datasets [/api/3/action/bulk_update_delete]

### Delete a list of datasets in an organisation [POST]

Make a list of datasets deleted

+ datasets (list of strings) – list of ids of the datasets to update
+ org_id (int) – id of the owning organization

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=bulk_update_public",
            "success": true,
            "result": null
        }


## Update CKAN site configuration [/api/3/action/config_option_update]

### Update specific configuration options [POST]

Allows modifications of some CKAN runtime-editable config options.

It takes arbitrary key, value pairs and checks the keys against the config options update schema. If some of the provided keys are not present in the schema a ValidationError is raised. The values are then validated against the schema, and if validation is passed, for each key, value config option:

* It is stored on the system_info database table
* The Pylons config object is updated.
* The app_globals (g) object is updated (this only happens for options explicitly defined in the app_globals module.

The following lists a key parameter, but this should be replaced by whichever config options want to be updated, eg:

    get_action('config_option_update)({}, {
        'ckan.site_title': 'My Open Data site',
        'ckan.homepage_layout': 2,
    })

Note: You can see all available runtime-editable configuration options calling the /api/3/action/config_option_list action.

Note: Extensions can modify which configuration options are runtime-editable. For details, check Making configuration options runtime-editable.

Warning: You should only add config options that you are comfortable they can be edited during runtime, such as ones you’ve added in your own extension, or have reviewed the use of in core CKAN.

+ key (string) – a configuration option key (eg ckan.site_title). It must be present on the update_configuration_schema

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=config_option_update",
            "success": true,
            "result": {
                "ckan.site_title": "CKAN site"
            }
        }

# Group Partial update functions

API functions for partial updates of existing data in CKAN.

The difference between the update and patch methods is that the patch will perform an update of the provided parameters, while leaving all other parameters unchanged, whereas the update methods deletes all parameters not explicitly provided in the data_dict.

## Patch a package [/api/3/action/package_patch]

### Patch a specific dataset [POST]

Selectively update a dataset (package). The user must be authorized to edit the dataset and the groups that it belongs to.

+ id (string) – the id or name of the dataset
+ title (string) – the title of the dataset. Optional
+ author (string) – the name of the dataset’s author. Optional
+ author_email (string) – the email address of the dataset’s author. Optional
+ maintainer (string) – the name of the dataset’s maintainer. Optional
+ maintainer_email (string) – the email address of the dataset’s maintainer. Optional
+ license_id (license id string) – the id of the dataset’s license, see /api/3/action/license_list for available values. Optional
+ notes (string) – a description of the dataset. Optional
+ url (string) – a URL for the dataset’s source. Optional
+ version (string, no longer than 100 characters) – Optional
+ state (string) – the current state of the dataset, e.g. 'active' or 'deleted', only active datasets show up in search results and other lists of datasets, this parameter will be ignored if you are not authorized to change the state of the dataset. Optional, default: 'active'
+ type (string) – the type of the dataset, IDatasetForm plugins associate themselves with different dataset types and provide custom dataset handling behaviour for these types. Optional
+ resources (list of resource dictionaries) – the dataset’s resources, see /api/3/action/resource_create for the format of resource dictionaries. Note: The complete resource list must be provided or else this update will be overwritten.
+ tags (list of tag dictionaries) – the dataset’s tags, see /api/3/action/tag_create for the format of tag dictionaries. Optional
+ extras (list of dataset extra dictionaries) – the dataset’s extras, extras are arbitrary (key: value) metadata items that can be added to datasets, each extra dictionary should have keys 'key' (a string), 'value' (a string). Optional
+ groups (list of dictionaries) – the groups to which the dataset belongs, each group dictionary should have one or more of the following keys which identify an existing group: 'id' (the id of the group, string), or 'name' (the name of the group, string), to see which groups exist call /api/3/action/group_list. Optional
+ owner_org (string) – the id of the dataset’s owning organization, see /api/3/action/organization_list or /api/3/action/organization_list_for_user for available values. Optional

+ Response 200 (application/json)

        
           "help": "http://demo.ckan.org/api/3/action/help_show?name=package_patch", 
           "result": {
               "author": "", 
               "author_email": "", 
               "creator_user_id": "ea7a7ce9-f004-4646-b59b-f81aa5b4518e", 
               "extras": [], 
               "groups": [], 
               "id": "79db7d1f-c0e1-44b7-a8ed-d8ea5da28600", 
               "isopen": true, 
               "license_id": "cc-by", 
               "license_title": "Creative Commons Attribution", 
               "license_url": "http://www.opendefinition.org/licenses/cc-by", 
               "maintainer": "", 
               "maintainer_email": "", 
               "metadata_created": "2015-09-30T15:30:40.092919", 
               "metadata_modified": "2015-09-30T21:08:59.094862", 
               "name": "another-dataset", 
               "notes": "Description of another dataset", 
               "num_resources": 1, 
               "num_tags": 0, 
               "organization": {
                   "approval_status": "approved", 
                   "created": "2015-09-08T12:21:40.893547", 
                   "description": "Organization description", 
                   "id": "a7acf343-0623-4be9-beb4-d24cbf3482f9", 
                   "image_url": "2015-09-08-122140.699178coffeefail.gif", 
                   "is_organization": true, 
                   "name": "an-organization", 
                   "revision_id": "0bc898a9-dcfa-4037-b786-99d323735617", 
                   "state": "active", 
                   "title": "An organization", 
                   "type": "organization"
               }, 
               "owner_org": "a7acf343-0623-4be9-beb4-d24cbf3482f9", 
               "private": false, 
               "relationships_as_object": [], 
               "relationships_as_subject": [], 
               "resources": [
                   {
                       "cache_last_updated": null, 
                       "cache_url": null, 
                       "created": "2015-09-30T15:31:03.994230", 
                       "datastore_active": false, 
                       "description": "Description of resource", 
                       "format": "CSV", 
                       "hash": "", 
                       "id": "dd56e61e-4cd7-4525-8d99-46fa8e3b9580", 
                       "last_modified": null, 
                       "mimetype": null, 
                       "mimetype_inner": null, 
                       "name": "Example resource", 
                       "package_id": "79db7d1f-c0e1-44b7-a8ed-d8ea5da28600", 
                       "position": 0, 
                       "resource_type": null, 
                       "revision_id": "3f39c35e-f171-444f-9924-a156b8b6756f", 
                       "size": null, 
                       "state": "active", 
                       "url": "http://example.com/dataset.csv", 
                       "url_type": null, 
                       "webstore_last_updated": null, 
                       "webstore_url": null
                   }
               ], 
               "revision_id": "fb6ce37f-3a6b-4915-a5b6-f7b92400d55f", 
               "state": "active", 
               "tags": [], 
               "title": "Dataset title", 
               "type": "dataset", 
               "url": "", 
               "version": ""
           }, 
           "success": true
        

## Patch a resource [/api/3/action/resource_patch]

### Patch a specific resource [POST]

Selectively update a resource.

+ id (string) – the id of the resource
+ url (string) – URL pointing to the resource. Must be provided, even if it should not be updated (should then include the same value)
+ name (string) – Name of the resource. Optional
+ description (string) – Description of the resource. Optional
+ mimetype (string) – Mimetype of resource. Optional
+ upload (multipart upload) - Resource file to upload (instead of a url). Optional

+ Response 200 (application/json)

        
           "help": "http://demo.ckan.org/api/3/action/help_show?name=resource_patch", 
           "result": {
               "cache_last_updated": null, 
               "cache_url": null, 
               "created": "2015-09-30T15:31:03.994230", 
               "description": "Description of resource", 
               "format": "CSV", 
               "hash": "", 
               "id": "dd56e61e-4cd7-4525-8d99-46fa8e3b9580", 
               "last_modified": null, 
               "mimetype": null, 
               "mimetype_inner": null, 
               "name": "Resource title", 
               "package_id": "79db7d1f-c0e1-44b7-a8ed-d8ea5da28600", 
               "position": 0, 
               "resource_type": null, 
               "revision_id": "f2c1fbdf-9790-41ae-a7f9-00a945421aaa", 
               "size": null, 
               "state": "active", 
               "title": "Resource title", 
               "url": "http://example.com/dataset.csv", 
               "url_type": null, 
               "webstore_last_updated": null, 
               "webstore_url": null
           }, 
           "success": true
        

## Patch a group [/api/3/action/group_patch]

### Patch a specific group [POST]

Selectively update a group.

+ id (string) – the id or name of the group
+ title (string) – the title of the group. Optional
+ description (string) – the description of the group. Optional
+ image_url (string) – the URL to an image to be displayed on the group’s page. Optional
+ type (string) – the type of the group, IGroupForm plugins associate themselves with different group types and provide custom group handling behaviour for these types Cannot be ‘organization’. Optional
+ state (string) – the current state of the group, e.g. 'active' or 'deleted', only active groups show up in search results and other lists of groups, this parameter will be ignored if you are not authorized to change the state of the group. Optional, default: 'active'
+ extras (list of dataset extra dictionaries) – the group’s extras, extras are arbitrary (key: value) metadata items that can be added to groups, each extra dictionary should have keys 'key' (a string), 'value' (a string), and optionally 'deleted'. Optional
+ packages (list of dictionaries) – the datasets (packages) that belong to the group, a list of dictionaries each with keys 'name' (string, the id or name of the dataset) and optionally 'title' (string, the title of the dataset)
+ groups (list of dictionaries) – the groups that belong to the group, a list of dictionaries each with key 'name' (string, the id or name of the group) and optionally 'capacity' (string, the capacity in which the group is a member of the group)
+ users (list of dictionaries) – the users that belong to the group, a list of dictionaries each with key 'name' (string, the id or name of the user) and optionally 'capacity' (string, the capacity in which the user is a member of the group)

+ Response 200 (application/json)

        
           "help": "http://demo.ckan.org/api/3/action/help_show?name=group_patch", 
           "result": {
               "approval_status": "approved", 
               "created": "2015-09-08T14:12:21.580163", 
               "description": "Description of the group", 
               "display_name": "Group title", 
               "extras": [], 
               "groups": [], 
               "id": "5fa0c44e-8ce5-4a4a-b1b4-c62f14388862", 
               "image_display_url": "http://demo.ckan.org/uploads/group/2015-09-08-141221.528384image.gif", 
               "image_url": "2015-09-08-141221.528384image.gif", 
               "is_organization": false, 
               "name": "a-dataset-group", 
               "package_count": 0, 
               "packages": [], 
               "revision_id": "46daf04f-c3ff-4287-a3d7-bf3f431a9665", 
               "state": "active", 
               "tags": [], 
               "title": "Group title", 
               "type": "group", 
               "users": []
           }, 
           "success": true
        

## Patch an organization [/api/3/action/organization_patch]

### Patch a specific organization [POST]

Selectively update an organization.

+ id (string) – the id or name of the organization
+ title (string) – the title of the organization. Optional
+ description (string) – the description of the organization. Optional
+ image_url (string) – the URL to an image to be displayed on the organization’s page. Optional
+ state (string) – the current state of the organization, e.g. 'active' or 'deleted', only active organizations show up in search results and other lists of organizations, this parameter will be ignored if you are not authorized to change the state of the organization. Optional, default: 'active'
+ extras (list of dataset extra dictionaries) – the organization’s extras, extras are arbitrary (key: value) metadata items that can be added to organizations, each extra dictionary should have keys 'key' (a string), 'value' (a string), and optionally 'deleted'. Optional
+ users (list of dictionaries) – the users that belong to the organization, a list of dictionaries each with key 'name' (string, the id or name of the user) and optionally 'capacity' (string, the capacity in which the user is a member of the organization)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=organization_patch", 
            "result": {
                "approval_status": "approved", 
                "created": "2015-09-08T20:20:15.147539", 
                "description": "Description for this other organisation", 
                "display_name": "The other organization", 
                "extras": [], 
                "groups": [], 
                "id": "a3c4b4b8-961d-45f4-8981-728653f36445", 
                "image_display_url": "http://demo.ckan.org/uploads/group/2015-09-08-202015.059790image.gif", 
                "image_url": "2015-09-08-202015.059790image.gif", 
                "is_organization": true, 
                "name": "another-organization", 
                "package_count": 1, 
                "packages": [
                    {
                        "author": "", 
                        "author_email": "", 
                        "creator_user_id": "ea7a7ce9-f004-4646-b59b-f81aa5b4518e", 
                        "extras": [], 
                        "groups": [], 
                        "id": "4e77badd-cd28-4825-97f1-9e2dffdbb002", 
                        "isopen": true, 
                        "license_id": "cc-by-sa", 
                        "license_title": "Creative Commons Attribution Share-Alike", 
                        "license_url": "http://www.opendefinition.org/licenses/cc-by-sa", 
                        "maintainer": "", 
                        "maintainer_email": "", 
                        "metadata_created": "2015-09-30T11:01:07.264156", 
                        "metadata_modified": "2015-09-30T15:46:39.531993", 
                        "name": "a-dataset", 
                        "notes": "A dataset description", 
                        "num_resources": 2, 
                        "num_tags": 0, 
                        "organization": {
                            "approval_status": "approved", 
                            "created": "2015-09-08T20:20:15.147539", 
                            "description": "Description for this other organisation", 
                            "id": "a3c4b4b8-961d-45f4-8981-728653f36445", 
                            "image_url": "2015-09-08-202015.059790image.gif", 
                            "is_organization": true, 
                            "name": "another-organization", 
                            "revision_id": "8c96920d-5676-4ba4-b465-2e48fb17efa3", 
                            "state": "active", 
                            "title": "A different organization", 
                            "type": "organization"
                        }, 
                        "owner_org": "a3c4b4b8-961d-45f4-8981-728653f36445", 
                        "private": true, 
                        "resources": [
                            {
                                "cache_last_updated": null, 
                                "cache_url": null, 
                                "created": "2015-09-30T11:35:00.174546", 
                                "description": "Description of other resource", 
                                "format": "CSV", 
                                "hash": "", 
                                "id": "248ed31b-bab5-4a0e-bb3e-9c03c2ffebf8", 
                                "last_modified": null, 
                                "mimetype": null, 
                                "mimetype_inner": null, 
                                "name": "Another resource", 
                                "package_id": "4e77badd-cd28-4825-97f1-9e2dffdbb002", 
                                "position": 0, 
                                "resource_type": null, 
                                "revision_id": "db723a9d-969e-43fd-8a50-828ac99dddcb", 
                                "size": null, 
                                "state": "active", 
                                "url": "http://example.com/another-dataset.csv", 
                                "url_type": null, 
                                "webstore_last_updated": null, 
                                "webstore_url": null
                            }, 
                            {
                                "cache_last_updated": null, 
                                "cache_url": null, 
                                "created": "2015-09-30T11:34:26.364593", 
                                "description": "A resource description", 
                                "format": "CSV", 
                                "hash": "", 
                                "id": "07889ab7-ae3b-44a3-8d68-c2b83b449541", 
                                "last_modified": null, 
                                "mimetype": null, 
                                "mimetype_inner": null, 
                                "name": "A resource", 
                                "package_id": "4e77badd-cd28-4825-97f1-9e2dffdbb002", 
                                "position": 1, 
                                "resource_type": null, 
                                "revision_id": "db723a9d-969e-43fd-8a50-828ac99dddcb", 
                                "size": null, 
                                "state": "active", 
                                "url": "http://example.com/dataset.csv", 
                                "url_type": null, 
                                "webstore_last_updated": null, 
                                "webstore_url": null
                            }
                        ], 
                        "revision_id": "44050580-37c1-4464-89ee-0f3255957272", 
                        "state": "active", 
                        "tags": [], 
                        "title": "The dataset title", 
                        "type": "dataset", 
                        "url": "", 
                        "version": ""
                    }
                ], 
                "revision_id": "275a2d08-f4d4-4d6e-bd9f-1fdddba8761e", 
                "state": "active", 
                "tags": [], 
                "title": "The other organization", 
                "type": "organization", 
                "users": []
            }, 
            "success": true
        }

# Group Removal functions

API functions for deleting data from CKAN.

## Delete a user [/api/3/action/user_delete]

### Delete a specific user [POST]

Delete a user. Only sysadmins can delete users.

+ id (string) – the id or usernamename of the user to delete

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=user_delete", 
            "result": null, 
            "success": true
        }

## Delete a package [/api/3/action/package_delete]

### Delete a dataset [POST]

Delete a dataset (package). This makes the dataset disappear from all web & API views, apart from the trash. You must be authorized to delete the dataset.

+ id (string) – the id or name of the dataset to delete

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=package_delete", 
            "result": null, 
            "success": true
        }

## Completely remove a package [/api/3/action/dataset_purge]

### Completely remove a dataset [POST]

Purge a dataset.

Warning: Purging a dataset cannot be undone!

Purging a database completely removes the dataset from the CKAN database, whereas deleting a dataset simply marks the dataset as deleted (it will no longer show up in the front-end, but is still in the db).

You must be authorized to purge the dataset.

+ id (string) – the name or id of the dataset to be purged

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=dataset_purge", 
            "result": null, 
            "success": true
        }

## Remove a resource [/api/3/action/resource_delete]

### Delete a specific resource [POST]

Delete a resource from a dataset. You must be a sysadmin or the owner of the resource to delete it.

+ id (string) – the id of the resource

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=resource_delete", 
            "result": null, 
            "success": true
        }

## Remove a resource view [/api/3/action/resource_view_delete]

### Delete a specific resource view [POST]

Delete a resource_view.

+ id (string) – the id of the resource_view

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=resource_view_delete", 
            "result": null, 
            "success": true
        }

## Remove resource view type [/api/3/action/resource_view_clear]

### Remove all resource views of a resource type [POST]

Delete all resource views, or all of a particular type.

+ view_types (list) – specific types to delete (optional)

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=resource_view_clear", 
            "result": null, 
            "success": true
        }

## Remove a group member [/api/3/action/member_delete]

### Remove a member of a group [POST]

Remove an object (e.g. a user, dataset or group) from a group. You must be authorized to edit a group to remove objects from it.

+ id (string) – the id of the group
+ object (string) – the id or name of the object to be removed
+ object_type (string) – the type of the object to be removed, e.g. package or user

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=member_delete", 
            "result": null, 
            "success": true
        }

## Remove a group [/api/3/action/group_delete]

### Delete a specific group [POST]

Delete a group. You must be authorized to delete the group.

+ id (string) – the name or id of the group

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=group_delete", 
            "result": null, 
            "success": true
        }

## Remove an organization [/api/3/action/organization_delete]

### Delete a specific organization [POST]

Delete an organization. You must be authorized to delete the organization.

+ id (string) – the name or id of the organization

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=organization_delete", 
            "result": null, 
            "success": true
        }

## Completely remove a group [/api/3/action/group_purge]

### Completely delete a group [POST]

Purge a group.

Warning: Purging a group cannot be undone!

Purging a group completely removes the group from the CKAN database, whereas deleting a group simply marks the group as deleted (it will no longer show up in the frontend, but is still in the db).

Datasets in the organization will remain, just not in the purged group.

You must be authorized to purge the group.

+ id (string) – the name or id of the group to be purged

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=group_purge", 
            "result": null, 
            "success": true
        }

## Completely remove an organization [/api/3/action/organization_purge]

### Completely delete a specific organization [POST]

Purge an organization.

Warning: Purging an organization cannot be undone!

Purging an organization completely removes the organization from the CKAN database, whereas deleting an organization simply marks the organization as deleted (it will no longer show up in the frontend, but is still in the db).

Datasets owned by the organization will remain, just not in an organization any more.

You must be authorized to purge the organization.

+ id (string) – the name or id of the organization to be purged

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=organization_purge", 
            "result": null, 
            "success": true
        }

## Stop following a user [/api/3/action/unfollow_user]

### Unfollow a user [POST]

Stop following a user.

+ id (string) – the id or name of the user to stop following

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=unfollow_user", 
            "result": null, 
            "success": true
        }

## Stop following a dataset [/api/3/action/unfollow_dataset]

### Unfollow a dataset [POST]

Stop following a dataset.

+ id (string) – the id or name of the dataset to stop following

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=unfollow_dataset", 
            "result": null, 
            "success": true
        }


## Delete a group member [/api/3/action/group_member_delete]

### Remove a specific user from a group [POST]

Remove a user from a group. You must be authorized to edit the group.

+ id (string) – the id or name of the group
+ username (string) – name or id of the user to be removed

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=group_member_delete", 
            "result": null, 
            "success": true
        }

## Delete an organizational member [/api/3/action/organization_member_delete]

### Remove a specific user from an organization [POST]

Remove a user from an organization. You must be authorized to edit the organization.

+ id (string) – the id or name of the organization
+ username (string) – name or id of the user to be removed

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=organization_member_delete", 
            "result": null, 
            "success": true
        }

## Stop following a group [/api/3/action/unfollow_group]

### Unfollow a group [POST]

Stop following a group.

+ id (string) – the id or name of the group to stop following

+ Response 200 (application/json)

        {
            "help": "http://demo.ckan.org/api/3/action/help_show?name=unfollow_group", 
            "result": null, 
            "success": true
        }
