# Customizing configurations

In this guide we look into different ways you can create custom configurations
for the [Community Solid Server (CSS)](https://github.com/CommunitySolidServer/CommunitySolidServer),
both what is needed to have a working configuration and some ways you can find the components you want to edit.

Before starting, you should have a basic grasp of how the CSS works.
The ["getting started" tutorial](getting-started.md) is a good place to get the basics down.

## Index

- [Customizing configurations](#customizing-configurations)
  * [Index](#index)
  * [Configurations in the Community Solid Server](#configurations-in-the-community-solid-server)
  * [Using the pre-defined imports](#using-the-pre-defined-imports)
  * [Finding the relevant component](#finding-the-relevant-component)
    + [Locking example](#locking-example)
    + [Template folder example](#template-folder-example)
    + [Alternative solutions to finding components](#alternative-solutions-to-finding-components)
  * [Overriding preset values](#overriding-preset-values)
  * [Adding components](#adding-components)
  * [Replacing components](#replacing-components)
  * [Conclusion](#conclusion)

## Configurations in the Community Solid Server

The CSS project is not a single implementation of a Solid server. 
Instead, it contains a collection of many components
that can be combined to create multiple variations of such a server.
These combinations are made using the dependency injection framework 
[Components.js](https://github.com/LinkedSoftwareDependencies/Components.js).
The project also comes with a set of Components.js 
[configurations](https://github.com/CommunitySolidServer/CommunitySolidServer/tree/main/config) 
that showcase how to do this.
These example configurations can also be used to quickly set up a server instance
without having to bother with configuration details yourself.
However, it is impossible to provide configurations for every possible combination,
as there are simply too many possible options.
This is why it will quite often be necessary to create your own configuration
to have a CSS instance with the features that you want.

## Using the pre-defined imports

The easiest way to create a custom configuration is to start from a default configuration
that most closely matches what you want, and adapt that one.
Below is a copy of one of those default configurations:
```json
{
  "@context": "https://linkedsoftwaredependencies.org/bundles/npm/@solid/community-server/^7.0.0/components/context.jsonld",
  "import": [
    "css:config/app/init/static-root.json",
    "css:config/app/main/default.json",
    "css:config/app/variables/default.json",
    "css:config/http/handler/default.json",
    "css:config/http/middleware/default.json",
    "css:config/http/notifications/all.json",
    "css:config/http/server-factory/http.json",
    "css:config/http/static/default.json",
    "css:config/identity/access/public.json",
    "css:config/identity/email/default.json",
    "css:config/identity/handler/default.json",
    "css:config/identity/oidc/default.json",
    "css:config/identity/ownership/token.json",
    "css:config/identity/pod/static.json",
    "css:config/ldp/authentication/dpop-bearer.json",
    "css:config/ldp/authorization/webacl.json",
    "css:config/ldp/handler/default.json",
    "css:config/ldp/metadata-parser/default.json",
    "css:config/ldp/metadata-writer/default.json",
    "css:config/ldp/modes/default.json",
    "css:config/storage/backend/file.json",
    "css:config/storage/key-value/resource-store.json",
    "css:config/storage/location/pod.json",
    "css:config/storage/middleware/default.json",
    "css:config/util/auxiliary/acl.json",
    "css:config/util/identifiers/suffix.json",
    "css:config/util/index/default.json",
    "css:config/util/logging/winston.json",
    "css:config/util/representation-conversion/default.json",
    "css:config/util/resource-locker/file.json",
    "css:config/util/variables/default.json"
  ]
}
```

This is a JSON-LD Components.js configuration.
It is special in that it only contains `@import` statements and doesn't define any components in its body.
This is done to provide a logical structure that groups certain clusters of components together,
so that if you want to change one specific feature or setting of the server you can look at that specific cluster
instead of having to look through all the available configuration files.
For many clusters there are multiple options available,
allowing you to choose between different settings for that cluster,
such as how the data should be stored or if account registration should be enabled.
All of these are described in the `config` folder of the GitHub project.
For example, you can find the descriptions of the `storage` clusters
[here](https://github.com/CommunitySolidServer/CommunitySolidServer/tree/main/config/storage).
There we can see that there are several options for how we want the server to store the data,
such as in memory or on the file system.

These descriptions exist for every import line in the above example.
You can quickly make a custom configuration that better suits your needs by copying the example above,
and for every line check the documentation for available options and pick those you prefer.

It is important that you do not remove any of the import lines in case you do not want a specific feature.
This might cause other parts of the configurations to fail
as they could depend on components being defined in the import you removed.
Several of the imports have a specific option to disable them that you can use instead.
We will cover what you can do if none of the options match your needs later in this guide.
Some options might change or new ones get added for a new major release of the server.
These will always be documented in the release notes.

There is also a configuration tool that provides a user interface for doing what we described above.
It can be found at <https://communitysolidserver.github.io/configuration-generator/>
and generates JSON-LD that you can use to start a CSS instance with the chosen options.

## Finding the relevant component

In case none of the import clusters completely support what you want,
this usually means you either want to modify or completely replace one or more components.
There are several things you can do to make the necessary changes,
which will be covered later in this guide,
but the first and main step is finding the component(s) that you want to change.
There is no exact way how to do this
and will require looking into the Components.js configuration files CSS provides,
So having some basic understanding of how these work and look like will help.

A first step that can be quite helpful is diving deeper into the configuration of the cluster
that contains the feature you want to change.
Chances are that configuration will contain the definition of the component you want to change.
In case there is no clear answer in there because there are many components, and you are not sure which one to edit,
or it is not clear what the parameters of a component do,
the [architecture](https://communitysolidserver.github.io/CommunitySolidServer/6.x/architecture/overview/)
or [API](https://communitysolidserver.github.io/CommunitySolidServer/6.x/docs/)
documentation might provide further information.

Below are some examples showing how you could discover where you need to make changes.
Note that this is not an exact science, 
and in case you get stuck you can always ask for help on the [Gitter](https://gitter.im/CommunitySolidServer/community)
or in the GitHub [repository](https://github.com/CommunitySolidServer/CommunitySolidServer/discussions).

### Locking example

Let's look at the use case of changing the timeout of the locking system.
One of the clusters in the example above is `"css:config/util/resource-locker/file.json"`,
which determines that the server will use a file-based locking system.
If we have a look at this
[file](https://github.com/CommunitySolidServer/CommunitySolidServer/blob/v6.0.0/config/util/resource-locker/file.json)
we see the following (some parts cut for brevity):

```json
{
  "@context": "https://linkedsoftwaredependencies.org/bundles/npm/@solid/community-server/^7.0.0/components/context.jsonld",
  "@graph": [
    {
      "comment": "Allows multiple simultaneous read operations. Locks are stored on filesystem. Locks expire after inactivity. This locker is threadsafe.",
      "@id": "urn:solid-server:default:ResourceLocker",
      "@type": "WrappedExpiringReadWriteLocker",
      "locker": {
        "@type": "PartialReadWriteLocker",
        ...
      },
      "expiration": 6000
    },
    {
      "@id": "urn:solid-server:default:CleanupInitializer",
      ...
    },
    {
      "@id": "urn:solid-server:default:CleanupFinalizer",
      ...
    }
  ]
}
```

We suggest having a look at JSON-LD and Components.js documentation and tutorials
to have a full understanding of what is going on in this file,
but we'll provide a short summary here which should help you along.
Every `@type` field corresponds to a TypeScript class in the CSS project.
`WrappedExpiringReadWriteLocker` is a class for which you can find the
[source code](https://github.com/CommunitySolidServer/CommunitySolidServer/blob/v6.0.0/src/util/locking/WrappedExpiringReadWriteLocker.ts)
in the CSS repository.
A block with this field in it will tell Components.js that it should create an instance of this class if it finds it.
The `@id` field is a unique identifier that we create, so we can reference this instance in different locations.
Multiple blocks with the same `@id` value reference the same instantiation,
so if there are multiple blocks with the same `@id`,
Components.js will only create 1 instance and use its reference in all those locations.
This field will also allow us later on to reference the object of which we want to change a value.
All the other values are parameters for the constructor of the class (except for `comment`).

Now to get back to how we find the component we want to edit.
There are several components here, but one of them is of a locker type, has a description saying it is used for locks,
and, most importantly, has an `expiration` parameter.
The [API documentation](https://communitysolidserver.github.io/CommunitySolidServer/6.x/docs/classes/WrappedExpiringReadWriteLocker.html)
of this class also states

> Wraps around an existing ReadWriteLocker and adds expiration logic to prevent locks from getting stuck.

So we can be quite sure this is the expiration we want to edit.
How to exactly do this we will show in the override section further below.

### Template folder example

To change the templates that are used during pod creation we use a similar tactic as in the locking example above.
The default config has an import `"css:config/identity/pod/static.json"`,
which determines how pods creation works.
That [file](https://github.com/CommunitySolidServer/CommunitySolidServer/blob/v6.0.0/config/identity/pod/static.json)
has the following contents:

```json
{
  "@context": "https://linkedsoftwaredependencies.org/bundles/npm/@solid/community-server/^7.0.0/components/context.jsonld",
  "import": [
    "css:config/identity/pod/resource-generators/templated.json"
  ],
  "@graph": [
    {
      "comment": "Stores all new resources for a pod in the default resource store under the generated identifier.",
      "@id": "urn:solid-server:default:PodManager",
      "@type": "GeneratedPodManager",
      "store": { "@id": "urn:solid-server:default:ResourceStore" },
      "resourcesGenerator": { "@id": "urn:solid-server:default:ResourcesGenerator" }
    }
  ]
}
```

This does not seem to have anything related to the templates used, unfortunately.
It does have an import though: `"css:config/identity/pod/resource-generators/templated.json"`.
This means that
[file]((https://github.com/CommunitySolidServer/CommunitySolidServer/blob/v6.0.0/config/identity/pod/resource-generators/templated.json))
is also related to the pod management cluster.
It contains the following data:

```json
{
  "@context": "https://linkedsoftwaredependencies.org/bundles/npm/@solid/community-server/^7.0.0/components/context.jsonld",
  "@graph": [
    {
      "comment": "Generates pods based on the templates in the corresponding folder.",
      "@id": "urn:solid-server:default:PodResourcesGenerator",
      "@type": "StaticFolderGenerator",
      "templateFolder": "@css:templates/pod",
      "resourcesGenerator": { "@id": "urn:solid-server:default:TemplatedResourcesGenerator" }
    },
    ...
  ]
}
```

Similarly as in the locking example, the description seems to indicate we are at the right place
and the `"templateFolder": "@css:templates/pod"` looks like the parameter we want to change.

### Alternative solutions to finding components

It might not always be possible to find the component that we want using the technique from the examples above.
An alternative solution is to run a search over all the configurations available
looking for terms that are relevant for what you need.
For example, you might have found the relevant class using the API documentation or by browsing the source code.
In that case you could look where in Components.js there is a `@type` field with that class
that is responsible for the instantiation.
Another case is if you know a value to look for.
In the template example you might have found out that the CSS pod templates are in the `templates/pod` folder.
When looking for that string you would end up finding the file above.
In the locking example, you could even just look for all configuration files that contain the word `"lock"`
and see which one looks most relevant.

Of course, none of these solutions are ideal and will not always result in you finding what you need,
which is why asking what needs to change is always a solution.

## Overriding preset values

Sometimes the provided configuration options almost completely fulfill your needs,
but there is a small change that you want to do.
For example, you want locks to have a longer timeout period,
or you want to use a different folder of templates when a new pod gets created.
Doing this is quite easy actually: you can make use of the Components.js 
[override](https://componentsjs.readthedocs.io/en/latest/configuration/configurations/overrides/) feature.

Going back to the lock expiration example of above, the following config could be used to override the value:

```json
{
  "@context": "https://linkedsoftwaredependencies.org/bundles/npm/@solid/community-server/^7.0.0/components/context.jsonld",
  "@graph": [
    {
      "@type": "Override",
      "overrideInstance": { "@id": "urn:solid-server:default:ResourceLocker" },
      "overrideParameters": {
        "@type": "WrappedExpiringReadWriteLocker",
        "expiration": 10000
      }
    }
  ]
}
```

The above configuration tells Components.js to change the `expiration` value
of the `urn:solid-server:default:ResourceLocker` to 10000 (ms).
The `"@type": "WrappedExpiringReadWriteLocker"` value is repeated
as Components.js requires this in every block where we define parameters.

Since version 5.1.0 the CSS allows having multiple values for the `-c` parameter,
so you could start the server with `npx @solid/community-server -c @css:config/file.json my-override-config.json`,
which would cause Components.js to merge the default config with your override addition before starting the server.
This will result in a server which is the same as any started with the `@css:config/file.json` configuration,
except it has a different lock expiration time.

## Adding components

Some import clusters in the provided configurations only have a default value and no alternatives.
The main goal of these is to inform you of their existence.
If you have a look at their contents,
you will usually find an array or map of components to which you can append your own custom components.

An example of this is the `"css:config/util/representation-conversion/default.json"` import in the default configurations.
It contains, among others, the following component:

```json
{
  "comment": "Automatically finds a path through a set of converters from one type to another.",
  "@id": "urn:solid-server:default:ChainedConverter",
  "@type": "ChainedConverter",
  "converters": [
    { "@id": "urn:solid-server:default:ContentTypeReplacer" },
    { "@id": "urn:solid-server:default:RdfToQuadConverter" },
    { "@id": "urn:solid-server:default:QuadToRdfConverter" },
    { "@id": "urn:solid-server:default:ContainerToTemplateConverter" },
    { "@id": "urn:solid-server:default:ErrorToJsonConverter" },
    { "@id": "urn:solid-server:default:ErrorToQuadConverter" },
    { "@id": "urn:solid-server:default:ErrorToTemplateConverter" },
    { "@id": "urn:solid-server:default:MarkdownToHtmlConverter" },
    { "@id": "urn:solid-server:default:FormToJsonConverter" }
  ]
}
```

This component is responsible for all the content negotiation in the server.
The class that contains this array combines these converters to go from the actual data type to the requested type.
For example, if an error gets thrown and an `Accept: text/turtle` was part of the request,
the `ErrorToQuadConverter` will first convert the error to quads,
and then the `QuadToRdfConverter` will serialize those quads to turtle.
If you want your server to also be able to convert quads to HTML,
you could create a new `QuadsToHtmlConverter` and add it to this list.

As mentioned before, Components.js combines all blocks with the same identifier into a single component.
When doing that, if it finds multiple values for the same parameter and all those values are arrays,
it will combine them into a single larger array.
Key/value objects are treated similarly.
We can thus append our own converter with the following configuration:

```json
{
  "@context": [
    "https://linkedsoftwaredependencies.org/bundles/npm/@solid/community-server/^7.0.0/components/context.jsonld",
    "https://linkedsoftwaredependencies.org/bundles/npm/my-custom-package/^1.0.0/components/context.jsonld"
  ],
  "@graph": [
    {
      "@id": "urn:solid-server:default:ChainedConverter",
      "@type": "ChainedConverter",
      "converters": [
        { 
          "@type": "MyCustomConverter"
        }
      ]
    }
  ]
}
```

We tell Components.js here to add a new component of type `MyCustomConverter`
to the array that the component `urn:solid-server:default:ChainedConverter`
receives as input for its `converters` parameter. 
This can then be combined with a default configuration as in the previous example,
with a command such as `npx @solid/community-server -c @css:config/file.json my-override-config.json`,
and will set up the converters so yours is included.

More information on how to create a custom component can be found [here](https://github.com/CommunitySolidServer/hello-world-component).

## Replacing components

Sometimes an override or addition is not sufficient to make the changes you need.
While you can use a Components.js override to change the class type associated with an identifier,
you might, for some reason, need to make even more drastic changes.
In this case, the only option remaining is to write your own configuration, that contains the new components.
The configuration clusters provided by the CSS project can still be used, so you don't have to rewrite everything.

The idea is that you reuse all the clusters,
except for those that contain the definitions of the components that need to be replaced.
Of the components that you do replace,
it is important that you provide new definitions for the identifiers that are referenced in other clusters.
There is no strict rule about which identifiers these are,
the best way to find out is to do a search for them and see if they also occur outside the cluster you're replacing.

Let's go back to our locking example from before and assume we want to completely replace the locking system.
We need to create a configuration that excludes the `util/resource-locker` cluster
and provides a new implementation for the relevant identifiers found within.

```json
{
  "@context": "https://linkedsoftwaredependencies.org/bundles/npm/@solid/community-server/^7.0.0/components/context.jsonld",
  "import": [
    "css:config/app/init/static-root.json",
    "css:config/app/main/default.json",
    "css:config/app/variables/default.json",
    "css:config/http/handler/default.json",
    "css:config/http/middleware/default.json",
    "css:config/http/notifications/all.json",
    "css:config/http/server-factory/http.json",
    "css:config/http/static/default.json",
    "css:config/identity/access/public.json",
    "css:config/identity/email/default.json",
    "css:config/identity/handler/default.json",
    "css:config/identity/oidc/default.json",
    "css:config/identity/ownership/token.json",
    "css:config/identity/pod/static.json",
    "css:config/ldp/authentication/dpop-bearer.json",
    "css:config/ldp/authorization/webacl.json",
    "css:config/ldp/handler/default.json",
    "css:config/ldp/metadata-parser/default.json",
    "css:config/ldp/metadata-writer/default.json",
    "css:config/ldp/modes/default.json",
    "css:config/storage/backend/file.json",
    "css:config/storage/key-value/resource-store.json",
    "css:config/storage/location/pod.json",
    "css:config/storage/middleware/default.json",
    "css:config/util/auxiliary/acl.json",
    "css:config/util/identifiers/suffix.json",
    "css:config/util/index/default.json",
    "css:config/util/logging/winston.json",
    "css:config/util/representation-conversion/default.json",

    "css:config/util/variables/default.json"
  ],
  "@graph": [
    {
      "@id": "urn:solid-server:default:ResourceLocker",
      "@type": "MyCustomLocker",
      "parameterA": "valueB"
    }
  ]
}
```

The empty line is to visually indicate we are excluding the `util/resource-locker` cluster.
We also provide a new implementation for the `urn:solid-server:default:ResourceLocker` identifier.
Doing a search on the configurations we can see this identifier occurring multiple times in other clusters,
which makes sense since it's the identifier that will be referenced if a locker is required.
This way we know this is the identifier we want to replace.
All the other identifiers we can find in the `util/resource-locker` configurations
are either references to components that are defined somewhere else,
or are not referenced outside that file,
so we don't have to provide replacements for those.

## Conclusion

There are several tips and hints in this guide on how you can potentially customize a CSS instantiation,
but they all require looking around a bit in the configurations
and reaching some understanding about that specific part of the architecture.
While we would also love to have more comprehensive tooling around this,
there is currently no great alternative,
so simply asking for help on the [Gitter](https://gitter.im/CommunitySolidServer/community)
or in the GitHub [repository](https://github.com/CommunitySolidServer/CommunitySolidServer/discussions)
is always a valid solution.
