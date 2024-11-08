# Autoarchive unused slack channels

This is a fork of [Symantec/slack-autoarchive](https://github.com/Symantec/slack-autoarchive). We forked the repository and made the following changes:

* Migrated to the new Slack API (using the changes from from https://github.com/Symantec/slack-autoarchive/pull/51)
* Improved the `README.md`
* Other small improvements & fixes (see [ce84d1](https://github.com/CDTM/slack-autoarchive/commit/ce84d1baf96ea2e6e618adf8819fa39c0d7fea81), [8209175](https://github.com/CDTM/slack-autoarchive/commit/8209175827478a7042ad94ec02eb45c8757593d1), [914b0cc](https://github.com/CDTM/slack-autoarchive/commit/914b0cc7404beb7753bdbb33ea38aa6ec174dd4d)).

## Requirements

- python3
- Install requirements.txt ( `pip install -r requirements.txt` )
- An [OAuth token](https://api.slack.com/docs/oauth) from a [Slack app](https://api.slack.com/apps/) on your workspace that has the following permission scopes:
  - `channels:history`
  - `channels:read`
  - `channels:manage`
  - `channels:join`
  - `chat:write`
  - `chat:write.customize`
  - `chat:write.public`

Here is quick tutorial on how to configure a slack app and get the OAuth token:

https://github.com/user-attachments/assets/1307c715-d44b-4706-80b5-4e9c8ffc8bac

## Example Usages

The `SLACK_TOKEN` must be exposed as a environment variable before running your script. By default, the script will do a `DRY_RUN`. To perform a non-dry run, specify `DRY_RUN=false` as an environment variable as well. See sample usages below.
```
# Run the script in dry run archive mode...This will output a list of channels that will be archived.
SLACK_TOKEN=<TOKEN> python slack_autoarchive.py

# Run the script in active archive mode...THIS WILL ARCHIVE CHANNELS!
DRY_RUN=false SLACK_TOKEN=<TOKEN> python slack_autoarchive.py
```

## Environment variables

| **Name**               | **Description**                                                                                 | **Default**       | **Example Usage**            |
|-----------------------|-------------------------------------------------------------------------------------------------|-------------------|-----------------------------|
| `ADMIN_CHANNEL`       | The ID of the Slack channel where notifications about the reaper's actions will be sent.        | `''` (empty)      | `C12345678`                 |
| `DAYS_INACTIVE`       | The number of days a channel must be inactive before it is eligible for archiving.              | `60`              | `60`                        |
| `MIN_MEMBERS`         | The minimum number of members a channel must have to be exempt from archiving.                  | `0`               | `5`                         |
| `DRY_RUN`             | Flag to indicate whether the script should run in dry mode (no actual changes made) or not.     | `true`            | `true` or `false`           |
| `SLACK_TOKEN`         | The Slack API token required to authenticate the bot performing the archiving.                  | `''` (empty)      | `xoxb-1234-5678-abcdef`     |
| `WHITELIST_KEYWORDS`  | Comma-separated keywords; channels containing these in their name will not be archived.         | `''` (empty)      | `important,permanent`       |
| `SLACK_SKIP_PURPOSE`  | A keyword used in channel purposes to mark channels that should not be archived.                | `%noarchive`      | `%noarchive`                |

## How can I exempt my channel from being archived?

You can add the string `%noarchive` to your channel purpose or topic. There is also a whitelist file or env variable if you prefer.

## What channels will be archived

A channel will be archived by this script if it doesn't meet any of the following criteria:

- Has a messages in the past 60 days (use `DAYS_INACTIVE` environment variable to change this)
- Is whitelisted. A channel is considered to be whitelisted if the channel name contains keywords in the `WHITELIST_KEYWORDS` environment variable. Multiple keywords can be provided, separated by comma.

## What happens when a channel is archived by this script

- *Don't panic!* It can be unarchived by following [these instructions](https://get.slack.help/hc/en-us/articles/201563847-Archive-a-channel#unarchive-a-channel). However all previous members would be kicked out of the channel and not be automatically invited back.
- A message will be dropped into the channel saying the channel is being auto archived because of low activity.
- You can always whitelist a channel if it indeed needs to be kept despite meeting the auto-archive criteria.

## Custom archive messages

Just before a channel is archived, a message will be sent with information about the archive process. The default message is:

> This channel has had no activity for {} days. It is being auto-archived. If you feel this is a mistake you can <https://slack.com/archives/archived|unarchive this channel> to bring it back at any point. In the future, you can add '%noarchive' to your channel topic or purpose to avoid being archived. This script was run from this repo: https://github.com/Symantec/slack-autoarchive

To provide a custom message, simply edit `templates.json`.

## Contribute

In case you want to contribute to this project, please make sure your changes work correctly. Currently, there are no tests for this project. This means that you will have to test your changes manually.

Follow these steps to test your changes:

1. Have a playground Slack workspace where you can test the script.
2. Create a new slack app and generate a new OAuth token.
3. Run the script with your new OAuth token and make sure it works as expected.
   1. Make sure to have some channels that meet the criteria for archiving and some that don't.

## Docker

In case you want to use docker to run the script, here is how you can do it:

1. First build the docker image (in the root of the project): `docker build --tag autoarchive .`
2. Run the container (`DRY_RUN` is set to true by default): `docker run -e SLACK_TOKEN=<YOUR_AWESOME_TOKEN> autoarchive`
3. If your ready to archive run: `docker run -e SLACK_TOKEN=<YOUR_AWESOME_TOKEN> -e DRY_RUN=false autoarchive`
