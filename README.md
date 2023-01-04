# GitHub Action: JSON Feed to Mastoson

A GitHub Action that creates messages (toots) on your Mastodon account from JSON Feed items.

This should be a simple way to POSSE — [Publish (on your) Own Site, Syndicate Elsewhere](https://indieweb.org/POSSE) — content from your blog to your Mastodon account.

## Example usage

I recommend to try first with an action requiring a manual action, to test the settings.

Here's a minimal version, with only required inputs:

```yaml
name: Create toots from JSON Feed items
on:
  workflow_dispatch:

jobs:
  JSONFeed2Mastodon:
    runs-on: ubuntu-latest

    steps:
      # Checkout the repository to restore previous cache
      - name: Checkout
        uses: actions/checkout@v3

      # Look for new toots to create from items in the JSON feed
      - name: JSON Feed to Mastodon
        uses: nhoizey/github-action-jsonfeed-to-mastodon@v1
        with:
          feedUrl: "https://example.com/feed.json"
          mastodonInstance: "https://mastodon.social"
          mastodonToken: ${{ secrets.MASTODON_TOKEN }}

      # Push changes in the cache files to the repository
      # See https://github.com/stefanzweifel/git-auto-commit-action#readme
      - name: Commit and push
        uses: stefanzweifel/git-auto-commit-action@v4
```

You can then enhance your action with a schedule as defined in GitHub's [events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule), for example to automate creation of a toot every Monday at 8am:

```yaml
name: Create toots from JSON Feed items
on:
  schedule:
    # Run the action every Monday at 8am
    # See https://crontab.guru/#0_8_*_*_1
    - cron: "0 8 * * 1"
  workflow_dispatch:

jobs:
  JSONFeed2Mastodon:
    runs-on: ubuntu-latest

    steps:
      # Checkout the repository to restore previous cache
      - name: Checkout
        uses: actions/checkout@v3

      # Look for new toots to create from items in the JSON feed
      - name: JSON Feed to Mastodon
        id: createToot
        uses: nhoizey/github-action-jsonfeed-to-mastodon@v1
        with:
          feedUrl: "https://example.com/feed.json"
          mastodonInstance: "https://mastodon.social"
          mastodonToken: ${{ secrets.MASTODON_TOKEN }}

      # Push changes in the cache files to the repository
      # See https://github.com/stefanzweifel/git-auto-commit-action#readme
      - name: Commit and push
        uses: stefanzweifel/git-auto-commit-action@v4
```

## Settings ("Inputs" in GitHub Action language)

There are 3 required **inputs**, used in the examples above, but also some optional inputs — with default values — to fine tune when and how toots are created:

| input                | required? | default                               | description                                                                                              |
| -------------------- | --------- | ------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `feedUrl`            | Yes       |                                       | URL of the JSON Feed to fetch                                                                            |
| `mastodonInstance`   | Yes       |                                       | The root URL of the Mastodon instance where the toot should be created                                   |
| `mastodonToken`      | Yes       |                                       | Your access token for the Mastodon API, get it from `/settings/applications/new` on your instance        |
| `nbTootsPerItem`     | No        | 1                                     | Number of toots that can be created from the same item                                                   |
| `globalDelayToots`   | No        | 1440 (1 day)                          | Delay (in minutes) between any toot from this feed                                                       |
| `delayTootsSameItem` | No        | 129600 (90 days)                      | Delay (in minutes) between any toot for the same item from this feed (used only if `nbTootsPerItem > 1`) |
| `cacheDirectory`     | No        | `.cache`                              | Path to the directory where cache files are stored                                                       |
| `cacheFile`          | No        | `jsonfeed-to-mastodon.json`           | Name of the JSON file caching data from the feed and toots                                               |
| `cacheTimestampFile` | No        | `jsonfeed-to-mastodon-timestamp.json` | Name of the JSON file caching the timestamp of the last toot                                             |

## Outputs

The action sets an [**output** that you can use in following steps of your own action](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#outputs-for-docker-container-and-javascript-actions):

| output    | description                      |
| --------- | -------------------------------- |
| `tootUrl` | URL of the toot that was created |

## Cache usage

There are 2 JSON files in the cache:

- `cacheFile` keeps a copy of all items from the feed, and for each item
  - the timestamp of the last toot created for this item
  - the list of URLs for these toots
- `cacheTimestampFile` keeps track of the timestamp of the last toot create by the action

Make sure to have steps "Checkout" and "Commit and push" in your action, this is how the cache files are synchronized each time the action runs.

The cache prevents creating the same toot multiple times if you set `nbTootsPerItem` to 1 (which is the default).

If you set `nbTootsPerItem` to a value larger than 1, the action will randomly chose an item among the ones that have the least toots. In particular, any new item in the feed won't have existing toots, so it will be tooted first when the action runs.

## License

The scripts and documentation in this project are released under the [MIT License](LICENSE).
