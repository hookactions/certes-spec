# Full event flow examples

Throughout this spec I've shown some different examples in their own specific context of how certain features of Certes might be used. This page is dedicated to showing full event flow examples to hopefully clear up any outstanding usage questions.

## As Meetly, Subscribe to GitHub

For this example, I am a software developer at Meetly who wants to subscribe to events that GitHub is sending. At this time GitHub does not officially support Certes so I will use the community developed schemas.

1. Go to `mgmt.meetly.com`
  - This can be a locally hosted [Management UI](#todo) by Meetly or a third-party Management UI.
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
