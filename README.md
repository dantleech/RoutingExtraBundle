# Symfony CMF Routing Component [![Build Status](https://secure.travis-ci.org/symfony-cmf/Routing.png)](http://travis-ci.org/symfony-cmf/Routing)


This library extends the Symfony2 Routing component. Even though it has Symfony
in its name, it does not need the full Symfony2 framework and can be used in
standalone projects. We also provide a [Symfony2 bundle](https://github.com/symfony-cmf/RoutingExtraBundle).

This component provides a *chain router* that can chain several RouterInterface
instances one after the other. It uses a list of routers, ordered by a priority
value, the higher the priority number the earlier that router is asked to match
respectivly generate routes. One of the routers in that chain can of course be
the Symfony2 router instance so you can still use the standard way for some of
your routes.

Additionally, this component is meant to provide useful router implementations.
Currently, there is the *DynamicRouter* that routes based on a implemented
repository that provide Symfony2 Route objects. The repository can be
implemented using a database, for example with doctrine phpcr-odm or doctrine
orm.

**Note**: To use this component outside of the Symfony2 framework context, have
a look at the [Symfony2 Routing component](https://github.com/symfony/Routing)
to get a fundamental understanding of the component. CMF Routing just extends
the basic behaviour.


## Dependencies

This component uses [composer](http://getcomposer.org). It needs the
Symfony2 Routing component and the Symfony2 HttpKernel (for the logger
interface and cache warmup interface).

For the DynamicRouter you will need something to implement the
RouteRepositoryInterface with. We suggest using Doctrine as this allows to map
any class into a database.


## ChainRouter

The ChainRouter does not route anything on its own, but only loops through all
chained routers. Add your router instances with the ``add`` method, then try
to resolve routes with all added routers using the ``match`` method and
Please refer to the phpdoc comments on the public methods for details.

## Dynamic Router

This implementation of a router loads routes from a RouteRepositoryInterface.
This interface can be easily implemented with doctrine.
The router works with the base UrlMatcher and UrlGenerator classes and only
adds loading routes from the database and the concept of referenced content.

To instantiate a DynamicRouter, you need an implementation of the
RouteRepositoryInterface. See the [Symfony2 RoutingExtraBundle](https://github.com/symfony-cmf/RoutingExtraBundle)
document classes for an example.

You will want to create controller mappers that decide what controller will
be used to handle the request, to avoid hardcoding controller names into your
content.

### Match Process

The match method of the DynamicRouter does the following steps

* Ask the repository for Route documents that could match the requested url
* Build a route collection and let the UrlMatcher find a matching route
* If the defaults do not contain the field _controller, loop through the
    ControllerMapperInterface list to find the controller. If none of the
    mapper finds a controller, throw a ResourceNotFoundException
* If the route implements RouteObjectInterface and returns a non-null content,
    set it in the returned array with key ``_content``.


### RouteObjectInterface

Routes that implement this interface are linked to a content document.
All routes still need to extend the base class Symfony\Component\Routing\Route

### Redirections

You can build redirections with a RedirectRouteInterface document. It can
redirect either to an absolute URI, or to a named route that can be generated by
any router in the chain or to another Route object in the repository.
To handle it, you can make ControllerClassMapper map a suitable controller
to Symfony\Cmf\Component\Routing\RedirectRouteInterface

### Routes and locales

You can use the _locale default value in a route to create one route per locale
that all reference the same multilingual content.
The DynamicRouter respects the _locale when generating routes from content.
When resolving the route, the _locale gets into the request and is picked up
by the symfony locale system.

A note for PHPCR-ODM: Routes should never be translatable documents, as one
route document represents one single url, and serving several translations
under the same url is not a good idea.


### Customize

You can add more ControllerMapperInterface implementations if you have a case
not handled by the provided ones.

For more specific needs, have a look at DynamicRouter and see if you want to
extend it. You can also write your own routers to hook into the chain.

### Url generation

You can generate urls for your content in three ways:

* Either pass an implementation of RouteObjectInterface as parameter ``route``

* Or pass a content object as parameter ``content``

* Or supply an implementation of ContentRepositoryInterface and the id of the content as parameter ``content_id``

## Authors

* Filippo De Santis (p16)
* Henrik Bjornskov (henrikbjorn)
* Claudio Beatrice (omissis)
* Lukas Kahwe Smith (lsmith77)
* David Buchmann (dbu)
* [And others](https://github.com/symfony-cmf/Routing/contributors)

The original code for the chain router was contributed by Magnus Nordlander.
