# Blog analytics

The Hugo build emits PostHog pageview tracking only when the `POSTHOG_KEY` environment variable is set. Without it, no PostHog script is rendered.

## One-time setup

1. Create a Web project in PostHog and select the US region, unless the project is explicitly hosted in the EU region.
2. In the `joeolaoye/blog` GitHub repository, add an Actions secret named `POSTHOG_KEY` containing the project's **public Project API Key**.
3. Push to `main` or re-run the Build and deploy workflow.

Do not store a PostHog personal API key in GitHub Actions or the site. The project key is designed to be visible in browser code; privileged keys are not.

## Per-post view dashboard

In PostHog, create an Insights trend for the `$pageview` event and break it down by the current URL or pathname property. Filter to `/blog/posts/` to exclude navigation and archive pages, then save it as **Blog post views**.

The tracker creates anonymous pageview analytics only. It does not identify visitors, enable session replay, or collect custom form fields.
