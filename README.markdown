## Contacts

Fetch users' contact lists without asking them to provide their
passwords, as painlessly as possible.

Contacts provides adapters for:

 * Google
 * Yahoo!
 * Windows Live

## Basic usage instructions

First, register your application with the service providers you
require. Instructions below under "Setting up your accounts".

Next, configure your accounts with a .yml file. Refer to
``spec/config/contacts.yml`` for configuration format

If you are using this as a gem instead of a Rails plugin, make sure
to add a ``config/initializer/contacts.rb``

    require 'contacts'
    config = YAML.load_file("#{Rails.root}/config/contacts.yml")
    Contacts.configure(config[Rails.env])

Now, in your controller, create a consumer:

    consumer = Contacts::Google.new
    consumer = Contacts::Yahoo.new
    consumer = Contacts::WindowsLive.new
    # OR by parameter:
    # provider is one of :google, :yahoo, :windows_live
    consumer = Contacts.new_consumer(provider)

Now, direct your user to:

    consumer.authentication_url(return_url)

`return_url` is the page the user will be redirected to by the service
once authorization has been granted.

Only after calling ``authentication_url``, can you try to persist the
consumer object so you can grab the contacts once the user returns:

    session[:consumer] = consumer.serialize

Now in the request handler of the return_url above:

    consumer = Contacts::Google.deserialize(session[:consumer])
    consumer = Contacts::Yahoo.deserialize(session[:consumer])
    consumer = Contacts::WindowsLive.deserialize(session[:consumer])
    # OR by parameter:
    # provider is one of :google, :yahoo, :windows_live
    consumer = Contacts.deserialize_consumer(provider, session[:consumer])
    if consumer.authorize(params)
      @contacts = consumer.contacts
    else
      # handle error
    end

Here, `params` is the hash of request parameters that the user returns
with. `consumer.authorize` returns true if the authorization was
successful, false otherwise.

The list of `@contacts` are `Contacts::Contact` objects which have
these attributes:

 * name
 * emails

## Setting up your accounts

### Google

Set up your projects
[here](http://code.google.com/apis/accounts/docs/RegistrationForWebAppsAuto.html).

 * When redirecting to an unverified domain (e.g., localhost), the
   user sees a warning when authorizing access.

What this means:

 * You can use the same consumer key for development as production.

### Yahoo

Set up your projects [here](https://developer.apps.yahoo.com/projects)

 * When a project's domain is verified, you must redirect to it.
 * When a project's domain is not verified, you can redirect anywhere, and the
   user sees a warning when authorizing access.

What this means:

 * Set up separate production and development projects.
 * Verify the domain of your production project. You will only be able to
   redirect to your production domain when using this key.
 * Don't verify the domain of your development project. You will be able to
   redirect to any domain when using this key.

## Windows Live

Set up your projects [here](http://msdn.microsoft.com/en-us/library/cc287659.aspx).

 * There is no domain verification step.
 * The domain cannot be localhost, an IP address, or have a query string.
 * You must redirect to the project's domain, on any port, and the URL may not
   have a query string or fragment.
 * You must specify a privacy policy URL.

What this means:

 * Set up separate production and development projects.
 * For development, use a domain like myapp.local.
 * Map this domain to 127.0.0.1 in /etc/hosts or a local DNS server.
 * If you want to run your app on a different domain (e.g., localhost:3000),
   redirect the POST from Windows Live to a GET on the original domain. This
   ensures the popup window has the same origin as the opener page, in
   accordance with browser same origin policies.

## Your Own Provider

You're not limited to only Google, Yahoo and WindowsLive. Add your own
configuration in the .yml

    my_own_provider:
      appid: lorem
      secret: ipsum

Create your custom class under ``Contacts`` module before initialization,
e.g. by modifying your config/initializer/contacts.rb like this

    require 'contacts'
    module Contacts
      class MyOwnProvider
        # ...
      end
    end
    config = YAML.load_file("#{Rails.root}/config/contacts.yml")
    Contacts.configure(config[Rails.env])

## Copyright

Copyright (c) 2010 [George Ogata](mailto:george.ogata@gmail.com) See
LICENSE for details.

Derived from [Mislav's Contacts](http://github.com/mislav/contacts),
Copyright (c) 2009 [Mislav Marohnić](mailto:mislav.marohnic@gmail.com)
