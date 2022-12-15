# Farcaster alerts to Slack channel

This is a [Pipedream](https://pipedream.com) workflow to be alerted on new casts from [Farcaster](https://farcaster.xyz) that match a search term into Slack.

## Tools used

- Pipedream
- Webhooks
- Slack
- Alertcaster

## How it works

- Set up a term to search for in Alertcaster
- Alertcaster sends a webhook to Pipedream
- Pipedream triggers a workflow to begin based on payload from Alertcaster
- Payload is formatted nicely for Slack
- Pipedream sends message within designated Slack channel

## Set up

- Create an alert with [Alertcaster](https://alertcaster.xyz/)
  - Configure the "How often" with "As-it-happens"
  - Set to "Alert by webhook"
  - Input `webhook URL` from Alertcaster in [step 1](#1-trigger)
    - Click "**Test**" from within Alercaster to it's wired up to Pipedream properly
    - Select the event from Alertcaster within Pipedream once it arrives
  - Click "**Test workflow**" in Pipedream
- Add a Code step in Pipedream to set up the Slack Block Kit payload
  - Make sure the environment is at least `nodejs14.x`
  - Paste in the [code below](#step-2-code)
  - Run "**Test**"
- Add a Slack step with "**Send Message Using Block Kit**"
  - Use something like [this configuration](#step-3-send_block_kit_message)
  - Once the configuration is set, test it
- Once everything tests out ok, go ahead and click "**Deploy**" in the top right and you're in business!

## Pipedream Workflow

### Step 1: `trigger`

#### Event Data

Full HTTP request

#### HTTP Response

Return HTTP 200 OK

### Step 2: `code`

```js
export default defineComponent({
  async run({ steps, $ }) {

    // Grabs the cast body and linkifies @ mentions using Searchcaster results
    const fcTextInputString = steps.trigger.event.body.text;
    const regex = /@(\w+)/g;
    const mentionLinkString = "<https://searchcaster.xyz/search?username=$1|@$1>";
    const fcText = fcTextInputString.replace(regex, mentionLinkString);

    // Takes the caster username and turns it into a link
    const fcUsername = steps.trigger.event.body.author_display_name;
    const fcUserSearchUrl = `<https://searchcaster.xyz/search?username=${fcUsername}|@${fcUsername}>`;

    // Takes the cast merkle root and creates a link to the cast, which will deep link to the Farcaster app from Alertcaster
    const fcMerkleRoot = steps.trigger.event.body.hash;
    const markdownLink = `<http://fc.alertcaster.xyz/casts/${fcMerkleRoot}/null|Open Cast> or see more casts from ${fcUserSearchUrl}`

    // Creates a Slack Block Kit block payload
    const blocks = [
      {
        type: "section",
        text: {
          type: "mrkdwn",
          text: `${fcText}`,
        },
      },
      {
        type: "context",
        elements: [
          {
            type: "mrkdwn",
            text: `${markdownLink}`,
          },
        ],
      },
    ];

    return blocks;
  },
});
```

### Step 3: `send_block_kit_message`

#### Slack Account

Use your preferred Slack account

#### Bot username

Copy the code below to use the following format for the bot username: "Full Name (@username)".

```
{{steps.trigger.event.body.author_display_name}} (@{{steps.trigger.event.body.author_username}})
```

#### Icon (image URL)

This users the caster avatar URL for the bot icon.

```
{{steps.trigger.event.body.author_pfp_url}}
```

#### Include link to workflow

Set to false

#### Channel

Send the message to the channel of your choosing.

#### Blocks

```
{{steps.code.$return_value}}
```
