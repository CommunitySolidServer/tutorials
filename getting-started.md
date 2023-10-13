This tutorial uses the [Community Solid Server (CSS)](https://github.com/CommunitySolidServer/CommunitySolidServer)
to both provide an introduction to Solid server behaviour,
and an introduction to the CSS itself.

This tutorial was last tested with v7.0.0 of the CSS.

## Index
- [Index](#index)
- [Requirements](#requirements)
- [Community Solid Server](#community-solid-server)
  * [Getting started](#getting-started)
  * [Example Solid HTTP requests](#example-solid-http-requests)
    + [GETting resources](#getting-resources)
    + [POSTing new resources](#posting-new-resources)
    + [PUTting new data](#putting-new-data)
    + [PATCHing resources](#patching-resources)
    + [DELETE a resource](#delete-a-resource)
  * [Using a different configuration](#using-a-different-configuration)
    + [File backend](#file-backend)
- [Creating a pod](#creating-a-pod)
  * [WebID](#webid)
- [Web Access Control (WAC)](#web-access-control--wac-)
  * [Access Control List](#access-control-list)
  * [Access Control Policies](#access-control-policies)
- [Authentication and client applications](#authentication-and-client-applications)
  * [Authentication client](#authentication-client)
  * [Authentication summary](#authentication-summary)
- [Editing Community server configurations](#editing-community-server-configurations)
  * [Components.js](#componentsjs)
  * [Customizing imports](#customizing-imports)
    + [Disabling authorization](#disabling-authorization)
  * [Configuration tool](#configuration-tool)
  * [Rewriting configurations](#rewriting-configurations)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## Requirements

You will need the following to follow this tutorial:

* [Node.js](https://nodejs.org/en/)
  * Version 18 or higher.
* A text editor

It makes things easier to keep all the work contained in a specific folder.
For the rest of this tutorial we will always assume this is the current working directory unless otherwise noted.

## Community Solid Server
CSS is an open and modular implementation of the Solid [protocol](https://solidproject.org/TR/protocol).
It's easy to set up, and due to its modular nature many features can be configured,
some of which we will cover later.

More information on the server can be found in the [repository](https://github.com/CommunitySolidServer/CommunitySolidServer)
and its corresponding [documentation](https://communitysolidserver.github.io/CommunitySolidServer/).

### Getting started
There are several ways you can install the server.
In this tutorial we will make use of `npx`, which is a command you get when installing Node.js.

To start the server, run the following command:
```shell
npx @solid/community-server
```
Make sure to confirm when the command asks you to install.
This will then download and install the server in the `npx` cache.
Once the log message appears indicating that the server has started,
you can find your own Solid server at http://localhost:3000/.

> If you prefer, you can also install the server by cloning the git repository using the following commands.
> Afterwards, all `npx @solid/community-server` commands in this tutorial should be replaced with `npm start --`.
> ```shell
> git clone https://github.com/CommunitySolidServer/CommunitySolidServer.git
> cd CommunitySolidServer
> npm ci
> ```

### Example Solid HTTP requests
All interactions with the server happen through HTTP requests,
with each HTTP method having its own use.
Below are some common examples.
You run these using `curl` on the command line,
or by manually constructing the queries in an HTTP client such as [Hoppscotch](https://hoppscotch.io/).

#### GETting resources
GET is used to request resources:

```shell
curl http://localhost:3000/
```
```turtle
@prefix dc: <http://purl.org/dc/terms/>.
@prefix ldp: <http://www.w3.org/ns/ldp#>.
@prefix posix: <http://www.w3.org/ns/posix/stat#>.
@prefix xsd: <http://www.w3.org/2001/XMLSchema#>.

<> a <http://www.w3.org/ns/pim/space#Storage>, ldp:Container, ldp:BasicContainer, ldp:Resource;
    dc:modified "2022-01-11T12:00:06.000Z"^^xsd:dateTime;
    ldp:contains <index.html>.
```

Note that this is a turtle representation of the root container.
CSS supports content negotiation for many formats,
in case n-triples are preferred these can be requested by sending the corresponding Accept header for example.

```shell
curl -H "Accept: application/n-triples"  http://localhost:3000/
```
```
<http://localhost:3000/> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/pim/space#Storage> .
<http://localhost:3000/> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/ldp#Container> .
<http://localhost:3000/> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/ldp#BasicContainer> .
<http://localhost:3000/> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/ldp#Resource> .
<http://localhost:3000/> <http://purl.org/dc/terms/modified> "2022-01-11T12:00:06.000Z"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
<http://localhost:3000/> <http://www.w3.org/ns/ldp#contains> <http://localhost:3000/index.html> .
```

And in case the HTML representation is needed, as seen in the previous step,
you can send a `text/html` Accept header along.

#### POSTing new resources
Post is used to create new resources.
We use the `-v` flag in the examples here to see the response headers,
since the Location header returns the URL of this new resource.

```shell
curl -v -X POST -H "Content-Type: text/plain" -d "abc" http://localhost:3000/
```

```
Location: http://localhost:3000/823df3de-5766-4d2a-97d7-2a79a7093c57
```

The Slug header is used to recommend a name for the resource to the server:

```shell
curl -v -X POST -H "Slug: custom" -H "Content-Type: text/plain" -d "abc" http://localhost:3000/
```
```
Location: http://localhost:3000/custom
```

You can see this resource at http://localhost:3000/custom.
Note that the Slug header is a suggestion,
servers are not required to use it.

#### PUTting new data
PUT is used to write data to a specific location.
In case that location already exists, the data there is overwritten.
If not, a new resource is created.

```shell
curl -X PUT -H "Content-Type: text/turtle" -d "<ex:s> <ex:p> <ex:o>." http://localhost:3000/a/b/c
```
Doing a GET to http://localhost:3000/a/b/c shows that the resource was created.
Note that this single request also created the intermediate containers 
http://localhost:3000/a/ and http://localhost:3000/a/b/, which you can also access now.

As mentioned, you can also overwrite the data at a location:
```shell
curl -X PUT -H "Content-Type: text/turtle" -d "<ex:s2> <ex:p2> <ex:o2>." http://localhost:3000/a/b/c
```
This command completely replaces the data found at http://localhost:3000/a/b/c.

#### PATCHing resources
PATCH is used to update resources.
This is useful if you do not want to completely overwrite a resource with a PUT
and only want to make a smaller change.

All Solid servers are required to support [N3 Patch](https://solid.github.io/specification/protocol#n3-patch)
to modify RDF resources. 
The following request modifies the resource we created in the previous step.
```shell
curl -X PATCH -H "Content-Type: text/n3" --data-raw "@prefix solid: <http://www.w3.org/ns/solid/terms#>. _:rename a solid:InsertDeletePatch; solid:inserts { <ex:s3> <ex:p3> <ex:o3>. }." http://localhost:3000/a/b/c
```

The above command will append the triple `<ex:s3> <ex:p3> <ex:o3>.` to the RDF resource,
which can be verified by GETting http://localhost:3000/a/b/c.

While not officially in the Solid spec,
CSS also supports [SPARQL Update](https://www.w3.org/TR/sparql11-update/) for PATCH requests, 
as do many other Solid servers.

The following request does the same as the N3 Patch request above.

```shell
curl -X PATCH -H "Content-Type: application/sparql-update" -d "INSERT DATA { <ex:s3> <ex:p3> <ex:o3> }" http://localhost:3000/a/b/c
```

#### DELETE a resource
Finally, DELETE is used to remove resources.

```shell
curl -X DELETE http://localhost:3000/a/b/c
```

This completely removes the resource from the server.
The parent containers will remain,
so http://localhost:3000/a/ and http://localhost:3000/a/b/ still exist.

### Using a different configuration
The server we used so far has been running with a memory backend,
which means all data is lost when the server is stopped.
There are many ways to configure CSS and several default configurations are provided that can be used.
When starting the server without any parameters, the server will default to a memory-based configuration.

#### File backend
To have a more permanent storage solution, we will now start the server with a file backend.
First stop the server that is still running, 
then start it again with the following command:
```shell
npx @solid/community-server -c config/file.json -f .data
```
The above command starts the server with the `file.json` config,
one of the available [default configurations](https://github.com/CommunitySolidServer/CommunitySolidServer/tree/main/config),
which starts the server with a file backend.
The `-f .data` parameter tells the server we want all data to be stored in the `.data` folder. 

## Creating a pod
A common concept in the Solid ecosystem is that of a "pod".
There is no strict definition of what a pod is,
but it generally corresponds to a space on a Solid server that a user controls and uses for their own data.

To create a pod on our own server instance,
we first have to create an account.
Now that your server is running again,
go to `http://localhost:3000/`,
click the button to sign up for an account,
and fill in the email/password combination you want to use.

Afterwards you will be redirected to your account page.
Once there, you can press the "Create pod" button to create your own pod.
You can choose any name you want, but in this tutorial we will assume you chose **my-pod**.
Make sure to keep the button checked to use the WebID in the pod.

### WebID
In the previous step the server created several resources for you,
including a profile document with a WebID.
A [WebID](https://dvcs.w3.org/hg/WebID/raw-file/tip/spec/identity-respec.html)
is a way to identify yourself on the Web
and is a core concept when authenticating with Solid.
We make use of this a bit later in this tutorial.
After creating your pod, you can find your profile resource at http://localhost:3000/my-pod/profile/card,
with your WebID being http://localhost:3000/my-pod/profile/card#me.

## Web Access Control (WAC)
In the previous HTTP examples no authentication was required to modify any of the resources,
but in practice you probably want to control who is allowed to read and modify data.
CSS (and most other Solid servers) make use of 
[Web Access Control (WAC)](https://solid.github.io/web-access-control-spec/)
to restrict access to resources.

### Access Control List
Your WebID needs to be publicly accessible,
so services can confirm your identity,
but your other resources can have restricted access.
For example, you can see the root of your pod at http://localhost:3000/my-pod/,
but when you try to browse to the profile container http://localhost:3000/my-pod/profile/,
you receive a `401: Unauthorized` error.

We cover how to correctly authenticate in the next section,
but we can already have a look now why this is the case.
Since the server is now running in file mode,
we can cheat and have a look at the files on disk to see what exactly is causing this.

WAC uses Access Control List (ACL) resources to determine what is allowed.
In general, these resources have an `.acl` extension,
but you can always find the corresponding ACL in the response header when GETting a resource.
The ACL resource that determines access to the root of our pod is `http://localhost:3000/my-pod/.acl`.
This means that we can find our root ACL resource by looking for the `../.data/my-pod/.acl` file.
This file has the following contents:

```turtle
# Root ACL resource for the agent account
@prefix acl: <http://www.w3.org/ns/auth/acl#>.
@prefix foaf: <http://xmlns.com/foaf/0.1/>.

# The homepage is readable by the public
<#public>
    a acl:Authorization;
    acl:agentClass foaf:Agent;
    acl:accessTo <./>;
    acl:mode acl:Read.

# The owner has full access to every resource in their pod.
# Other agents have no access rights,
# unless specifically authorized in other .acl resources.
<#owner>
    a acl:Authorization;
    acl:agent <http://localhost:3000/my-pod/profile/card#me>;
    # Optional owner email, to be used for account recovery:
    acl:agent <mailto:test@example.com>;
    # Set the access to the root storage folder itself
    acl:accessTo <./>;
    # All resources will inherit this authorization, by default
    acl:default <./>;
    # The owner has all of the access modes allowed
    acl:mode
        acl:Read, acl:Write, acl:Control.
```

A full explanation of all triples can be found in the WAC specification,
but we give a short summary here.
The first block makes it so the root container (`acl:accessTo <./>`) 
is readable (`acl:mode acl:Read`) 
by all agents (`acl:agentClass foaf:Agent`).

The second block defines that the user identified as 
our WebID (`acl:agent <http://localhost:3000/my-pod/profile/card#me>`)
has full access (`acl:mode acl:Read, acl:Write, acl:Control`)
to both the root container (`acl:accessTo <./>`)
and all its recursively contained resources (`acl:default <./>`).

This explains why we could not access the profile container:
we did not identify ourselves as our WebID.

You might wonder why we could read the `/my-pod/profile/card` resource
even though the above ACL resource prevents all access recursively.
The reason for this is that WAC tries to find the ACL resource
closest to the resource that is being accessed.
When trying to access http://localhost:3000/my-pod/profile/card,
the server will first try to find the ACL resource http://localhost:3000/my-pod/profile/card.acl.
Only if that one is not found will it try look for ACL resources in the parent containers:
http://localhost:3000/my-pod/profile/.acl and eventually http://localhost:3000/my-pod/.acl.
If you look again at the files, you will note that there is a `.data/my-pod/profile/card.acl` file,
which allows full read access to the resource,
thereby making the profile fully readable for everyone.

### Access Control Policies
WAC is just one way to determine authorization on a Solid server.
An alternative is [Access Control Policies (ACP)](https://solid.github.io/authorization-panel/acp-specification/),
which has more flexibility in determining resource access.
How ACP works is not covered in this tutorial,
but CSS can be configured to use ACP instead of WAC for authorization.

## Authentication and client applications
As could be seen in the previous section,
it is important to authenticate with a specific WebID to access protected resources.
Solid makes use of OpenID and OAuth to support this.
All information related to this can be found in the
[Solid-OIDC specification](https://solid.github.io/solid-oidc/).

First we show an example of how authentication works,
after which we provide an explanation of why exactly this works.

### Authentication client
The de facto open source solution for authenticating with a Solid server is the 
[solid-client-authn](https://github.com/inrupt/solid-client-authn-js) library.
To authenticate you will need client that makes use of such a library to authenticate you.
One example of such a client is [Penny](https://gitlab.com/vincenttunru/penny),
of which you can find a running instance at https://penny.vincenttunru.com/.

You can insert the URL of your pod in the top bar of Penny to get an
[overview](https://penny.vincenttunru.com/explore/?url=http%3A%2F%2Flocalhost%3A3000%2Fmy-pod%2F)
of the resources located within.
When clicking the `profile/` container though, we will again be informed we do not have access.
To authenticate with Penny, press the "Connect Pod" button in the top right corner, 
enter the URL of your server, `http://localhost:3000`, and connect.
This will redirect you to your server,
where you will be asked to authorize this client to log in with your WebID.
After doing so, you will be redirected back to Penny.
There you will see you now can access the profile container.

### Authentication summary
The Solid-OIDC specification contains all the details,
but below is a simplified summary of what happened during authentication above.

 1. The client, Penny in this case, asks what your Identity Provider (IdP) is.
 2. The client redirects you to that IdP, so you can identify yourself.
 3. The IdP provides proof that identifies you as the owner of your WebID.
 4. The client adds this proof to its request when accessing a resource on the Solid server.
 5. The Solid server GETs the WebID to find which IdP is responsible for proving its identity.
 6. The Solid server asks that IdP if the provided proof is valid.
 7. The Solid server verifies if the provided WebID can access the requested resource, based on the ACL resources. 
 8. The Solid server returns the resource.

You might be wondering what this IdP is that has been mentioned several times.
This is a server that conforms to the Solid-OIDC specification and can identify a WebID.
Fortunately the CSS, besides implementing the Solid protocol,
also fully implements the expected IdP behaviour,
so it is both a Solid server and an IdP.

Step 5 mentions that your WebID contains which IdP is responsible.
You can see this by looking at your profile document http://localhost:3000/my-pod/profile/card#me.
It will contain the triple `:me solid:oidcIssuer <http://localhost:3000/>`.
This is the triple that identifies our CSS instance as a trusted IdP.

In this example the CSS instance contained the resource you were trying to access,
it contained your WebID, and it was your trusted IdP.
In practice these could all be different servers,
which could all be different CSS instances or all instances of different Solid implementations.

During this interaction,
the client did not care that we were running a CSS instance,
any kind of Solid server would have worked.
This is because the algorithm is based on the Solid specification,
and not on any specific implementation.

## Editing Community server configurations
At this point you can stop your CSS instance as we are going to have a look
at how to customize a CSS configuration.
Do not delete the `.data` folder yet though as we will reuse that data.
We already saw in a previous section that it is possible to configure the server
to have a memory backend or a file backend, but many more options are available.

### Components.js
When we wanted CSS to store data on disk,
we told it to use the configuration
[config/file.json](https://github.com/CommunitySolidServer/CommunitySolidServer/blob/main/config/file.json),
which you can find in the CSS source folder.
This is a [JSON-LD](https://json-ld.org/) file that mostly imports many other JSON-LD documents.
If you explored all the imports you would find out that these documents
link all classes together with their parameters.
This is done using [Components.js](https://github.com/LinkedSoftwareDependencies/Components.js),
a semantic dependency injection framework.
In this tutorial we only cover a few basics of how CSS uses this framework.
More detailed information can be found in the Components.js
[documentation](http://componentsjs.readthedocs.io/).

### Customizing imports
The imports you see in the configuration file
all refer to files found in subfolders of the `config` folder.
These folders and files have been structured in a such a way
that every import line corresponds to a specific feature that can be changed.

For example, if you look at the config that is used
when running the server with `npx @solid/community-server` (`config/default.json`)
you can see it has the following import:
```
"css:config/storage/backend/memory.json"
```
On the other hand, `config/file.json` has the following import instead:
```
"css:config/storage/backend/file.json"
```
Simply by changing that import in a configuration you can tell it what backend to use.
Every subfolder of `config` has a `README.md` document explaining what the available options are.
For example, if you have a look at
[config/storage/README.md](https://github.com/CommunitySolidServer/CommunitySolidServer/tree/main/config/storage)
you can see that it is also possible to have a SPARQL backend by changing the import to use `sparql.json`.
At the time of writing there are 31 import lines, so those are 31 features that can easily be customized.

As for why these imports use the `css` prefix we refer to the Components.js documentation
as this is based on the project settings.

#### Disabling authorization
As an example, we will create a new config that fully disables authorization.
This way we can read all our data without needing an authentication client.
This is not something you want to do in production,
but can be useful when running experiments during development.

To start, copy the `config/file.json` file to your root tutorial folder,
this is the same folder that should contain your `.data` folder,
and rename it to `unsafe.json`.
The import we want to change is the one that is responsible for authorization,
which is the `css:config/ldp/authorization/webacl.json` import.
Looking at the documentation for the 
[ldp configuration options](https://github.com/CommunitySolidServer/CommunitySolidServer/blob/main/config/ldp/README.md)
we can see that another option for authorization is `allow-all`,
so change that import line to be `css:config/ldp/authorization/allow-all.json` instead.
Now start the server with the new server as follows:
```shell
npx @solid/community-server -c unsafe.json -f .data
```
This starts the server again on http://localhost:3000/.
If you now browse to http://localhost:3000/my-pod/profile/
you can see the contents of that container,
while this was hidden when running our previous instance,
showing that authentication no longer checks requests.
Note also that all data is still there even though 
the server was stopped and restarted with a different configuration.

### Configuration tool
There is also a configuration tool that provides a user interface for doing what we described above.
It can be found at <https://communitysolidserver.github.io/configuration-generator/>
and generates JSON-LD that you can use to start a CSS instance with the chosen options.
The previous steps can be reproduced by going to the "Authorization" option in the link above,
and selecting the "Disable" option.

### Rewriting configurations
Sometimes more customization is needed than what changing the imports allow.
In those cases it will be necessary to create a configuration that is a combination
of specific imports and custom Components.js.
This requires more knowledge about the server and Components.js, so we will not go deeper into this.
A more extensive tutorial on this can be found [here](custom-configurations.md).
An example is the
[config/sparql-file-storage.json](https://github.com/CommunitySolidServer/CommunitySolidServer/blob/main/config/sparql-file-storage.json)
configuration, which uses a file backend for internal data such as accounts,
and a SPARQL backend for all standard Solid data.
This configuration removed a few imports,
and instead replaced their functionality by the objects found in the body of that configuration.
