# Who defines the schema?

There are two scenarios to determine who should define the schema. For the first version of Certes, only the "Producer defined schema" will be supported. That doesn't mean that we can't code to be aware of the eventual addition of the "Subscriber defined schema".

## Producer defined schema

When creating custom integrations, this is the most common use case: I want to receive events from a third party, the third party has defined what those events look like, I need to write code to handle that data format. Some examples of this are: GitHub, Calendly, Copper CRM, and Stripe.

For this use case, the producer of events will define the schema and the subscriber must write code to accommodate these foreign data structures. In order to set up a subscription, the subscriber will call the local "[Master API](/master-api)" such as `certes.EnsureSubscriptions(...)`. The Master API will then ensure authentication is set up for the specified service and create the subscriptions.

## Subscriber defined schema

For this use case, it is more or less an API defined by the subscriber of events. The best example of this is for chat applications such as Slack or Discord where you want to send custom messages to a channel. Slack and Discord define the format of the data they need to receive and then it's up to you to format your information in that way. This type of schema definition is yet to be fully defined and implemented, it will be the largest feature of the second version of Certes.

A possible implementation would be for the producer (Meetly/your company) to call the local "Master API" to ensure the subscription is set up, similar to the "Producer defined schema". The returned endpoint would then be saved in the Master API. You would then call your local gateway/event broker to send an outgoing event to the third party.

Again, this needs to be flushed out more before implementation in Certes v2.

## Companies not using Certes

Every company with webhooks will not adopt Certes overnight to produce events. This does not mean companies should have to wait for them to do this. There will be a community repository where anyone can add schemas for third-party events. In many of the examples in this specification you will see `community.certes.dev` which is where these schemas will live.
