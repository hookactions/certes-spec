# Full event flow examples

Throughout this spec I've shown some different examples in their own specific context of how certain features of Certes might be used. This page is dedicated to showing full event flow examples to hopefully clear up any outstanding usage questions.

## As Meetly, subscribe to GitHub

For this example, I am a software developer at Meetly who wants to subscribe to events that GitHub is sending. At this time GitHub does not officially support Certes so I will use the community developed schemas.

1. Go to `mgmt.meetly.com`
  - This can be a locally hosted [Management UI](/management-ui) by Meetly or a third-party Management UI.
1. Enter `events://community.certes.dev/github`
  - The Management UI will call the Master API to retrieve events for this URI & Namespace.
  - The namespace "github" is optional, but can be used when there are many namespaces available.
  - The Master API will cache the schema in Meetly's local Schema Registry
1. I can now see what events & schema are available to be used in the `github` namespace. I will also see a short explanation of how authentication works for this namespace
  - Authentication will be either OAuth or an `Authorization` header token.
1. Connect via OAuth to GitHub which will grant `community.certes.dev` GitHub api access.
1. I am now able to use all events in the `community.certes.dev/github` namespace
  ```go
  func init() {
    certes.EnsureSubscription(
      certes.Event("community.certes.dev/github/org/1/memberships", &gh.Opts{
        Org: "meetly",
      }),
      certes.Event("community.certes.dev/github/repo/1/push", nil),
    )
  }
  ```
1. I then update my code to handle these incoming events
  ```go
  func main() {
    certes.On("community.certes.dev/github/org/1/memberships", func(raw *certes.RawEvent) error {
      var event gh.PushEvent
      _ = raw.To(&event)
      // handle event
      return nil
    })
  }
  ```
1. Test & deploy!

## As GitHub, define available events

For this example, I am a company who wants to expose events via Certes so that my clients can subscribe to events.

1. Install and set up Certes (or use a third-party)
1. Define domain (e.g. `events://events.github.com`)
1. Upload protobuf event schema
  - Either via `mgmt.events.github.com` or the third-party Certes provider
1. Share the `events://events.github.com` URI with the world and developers!

## As Meetly, subscribe to GitHub (take 2)

For this example, I am a software developer at Meetly who wants to subscribe to events that GitHub is sending. GitHub now has publicly available, official, event schemas and `events://events.github.com` URI.

1. Go to `mgmt.meetly.com`
1. Enter `events://events.github.com`
  - A namespace could be specified, but since GitHub will likely have very few namespaces it's ok to omit.
1. See the available events and schema as well as a short explanation about how authentication to GitHub will work.
1. Connect via OAuth to GitHub
1. Update code to ensure subscriptions are created (see above code)
  - Request is sent to local Master API to ensure a subscription is set up for the specified URI via `certes.Init("events://events.meetly.com")`
  - Third-party event gateway sends subscription details to third-party Master API. The subscription details will contain the third-party's internal ID which was stored upon OAuth connection. The details also include the events and any options (e.g. org, repo, etc.).
  - The details also include the callback URI (`events://events.meetly.com`) which is where events will be sent.
1. Update code to handle incoming events (see above code)
1. Test & deploy!

## As GitHub, send an event to a subscriber

1. Define available events and their schema (see above)
1. Call `SendOutgoingEvent` when an event occurs for any entity (org, repo, etc.)
  - The Event Broker will filter these events and only send the events that are actively subscribed to
  - For active subscriptions that are found, the event is queued
  - The Event Consumer receives the event and sends it to the subscriber destination (e.g. `events://events.meetly.com`)
  - The Event Consumer handles HMAC signing

  ```go
  package main

  import (
    certes "github.com/hookactions/certes-sdk/go"
    pbv1 "./pb/v1"  // locally generated protobuf code
    pbv2 "./pb/v2"  // locally generated protobuf code
  )

  func init() {
    certes.Init("events://events.github.com") // GitHub's event gateway, local or hosted by 3rd party
  }

  func main() {
    meetlyGhId := "org_123"

    // Something in our code triggers a "push" event for Meetly
    certes.SendOutgoingEvent(meetlyGhId, &pbv1.PushEvent{
      Ref: "refs/tags/simple-tag",
      Before: "6113728f27ae82c7b1a177c8d03f9e96e0adf246",
      // ... data here related to "push"
    })
    certes.SendOutgoingEvent(meetlyGhId, &pbv2.PushEvent{
      Ref: "refs/tags/simple-tag",
      BeforeCommitSha: "6113728f27ae82c7b1a177c8d03f9e96e0adf246",  // versioning example of field name changing
      // ... data here related to "push"
    })
  }
  ```
