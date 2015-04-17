# Definition Unlocked

This `Definition` object is massively important in Symfony's DependencyInjection
container, and these guys are being built behind-the-scenes all over the place.
I'll show you how in a second.

Beyond using the class name and passing consructor arguments, there are a
bunch of other things that you can train a `Definition` object to do. For
example, what if you wanted the container to instantiate a service, but then
also call an extra method on it *before* passing the service back? 

For example, as soon as the logger is created, I want to log a message. To
do that, use `addMethodCall()`. The `Logger` class has a `debug` method on
it - pass that as the first argument. This is equivalent to `$logger->debug()`.
Then pass the array of arguments - we have just one, the message we want
to log: "The logger just got started":

[[[ code('82d2097b4b') ]]]

With this, the container will create the logger, call the `debug()` method
on it, and *then* pass it back. Test it!

```bash
php dino_container/roar.php
tail dino_container/dino.log
```

Brilliant! Now let's get harder. 

There are actually *two* ways to add handlers to `Logger`: via the constructor,
like we're doing now, OR by calling a `pushHandler()` method. Let's see if
we can create a *second* handler and hook it up with a method call.

Start by creating the new Definition - it'll be to a stream again, so re-use
the `StreamHandler` class. This handler will dump the output to the console.
To do that, call `setArguments()` like before, but this time, we'll pass
a single argument: `php://stdout`. Woops, and I'll fix my wrong variable name.
Now put it into the container with `setDefinition()` - we'll call it `logger.std_out_logger`.

[[[ code('37fb18e132') ]]]

The file *is* getting long and ugly - so we'll fix that in a minute.

To register this handler with the `logger` service, we could just add it
to the second constructor argument. But to prove we've got this mastered,
let's use `$loggerDefinition->addMethodCall()` and tell it to call a `pushHandler`
method. This takes *one* argument: the actual handler object. To pass in
that service we just added, create a new `Reference` and give it the service
name: `logger.std_out_logger`:

[[[ code('6b2c65e02f') ]]]

Open up the `Logger` class so we can see what's happening. Inside this class,
there's a `public pushHandler()` method that has one argument:

```php
// ...
// the Monolog\Logger class
class Logger implements LoggerInterface
{
    // ...
    
    public function pushHandler(HandlerInterface $handler)
    {
        // ...
    }
}
```

Our code says: call `pushHandler` on the `Logger` object and pass it the
service whose id is `logger.std_out_logger`. With any luck, log messages
will dump into our file *and* get printed out to the screen.

Let's do it!

```bash
php dino_container/roar.php
tail dino_container/dino.log
```

Hey, there's one message! But inside `dino.log`, we see *both*. So why did
only one message get printed to the screen? Actually, it's just a matter of
order: we're calling the `debug` method *before* pushing the second handler.
If you want to fix things, just rearrange the two method calls. Now it prints
both.

In the Symfony Framework, or Drupal for that matter, *every* service in the
container started as a humble `Definition` object. This object lets you tweak
your future service object in a bunch of other ways too, including things
like adding tags, a thing called a configurator, creating objects through
factories and a few other things. To find out all you can handle about those,
head over to Symfony.com: there's a *massive* section in the Components area.
You can basically earn you degree in DependencyInjection. 

Now, how does all of this translate to what we see inside of Symfony: yaml
files.