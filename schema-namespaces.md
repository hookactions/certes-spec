# Schema Namespaces

It may be useful for some companies to define events within a namespace. For example, GitHub will only send certain to events for repositories and certain events for organizations. These events may even share the same name but contain different or the same data. This is also discussed when "Subscribing to events" section in the [URI page](/uri#subscribing-to-events).

## The Default Namespace (`_`)

If a company does not want to or need to use namespaces, there will always be a default namespace which can be notated by an underscore: `_`. In a URI it would look like this: `events.meetly.com/_/1/my_event`.

## Custom Namespaces

Custom name spaces work similarly to an S3 bucket key in the sense that for the sake of a URI path, it acts like a folder, but internally it is stored as a single key. Let's look at some examples:

```
events.meetly.com/org/1/my_event => "org" 
events.meetly.com/user/1/my_event => "user"

events.meetly.com/org/user/1/my_event => "org/user"
events.meetly.com/foo/bar/baz/1/my_event => "foo/bar/baz"
```

Notice that the namespaces are just a single string, NOT an array or list. This should simplify storage of namespaces and events under them. See the "[Schema Registry](/schema-registry)" component to read more about this decision.
