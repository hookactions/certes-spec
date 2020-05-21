The producer configuration is independent of the subscriber configuration and vice-versa. These configurations should be created via Master API or Management UI. There are some generic retry strategies included with Certes:

1. Logarithmic back-off configured via: maximum retries (absolute number) or maximum time to try (e.g. 7 days)
1. Linear back-off configured via: maximum retries (absolute number) or maximum time to try (e.g. 7 days)

These rules are set at a global level and may be configured per event source (e.g. GitHub) or per event type (e.g. GitHub push).
