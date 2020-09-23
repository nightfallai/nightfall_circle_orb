# Nightfall DLP CircleCI Orb
![nightfalldlp](https://nightfall.ai/wp-content/uploads/2020/08/nightfall-dark-logo-tm-e1597263930794.png)
## [Nightfall](https://nightfall.ai) DLP Orb: A code review tool that protects you from committing sensitive information to your repositories.

The Nightfall DLP CircleCI Orb scans your code commits for sensitive information - like credentials & secrets, PII, credit card numbers & more - and posts review comments to your code hosting service automatically. The Nightfall DLP CircleCI Orb is intended to be used as a part of your CI to simplify the development process, improve your security, and ensure you never accidentally leak secrets or other sensitive information via an accidental commit.

## Example
Here's an example of the Nightfall DLP CircleCI Orb providing feedback on a Pull Request:

![nightfall-dlp-circle-orb-example](https://cdn.nightfall.ai/nightfall-code-scanner/circleci-github-comments.png)

Here's the CircleCI log output from the same run:

![nightfall-dlp-circle-ui-example](https://cdn.nightfall.ai/nightfall-code-scanner/circleci-orb-workflow-ui.png)

The orb runs when configured as a job in your CircleCI config:

```yaml
version: 2.1

orbs:
  nightfall_code_scanner: nightfall/nightfall_code_scanner@v1.0.0

workflows:
  test_scanner:
    jobs:
      - nightfall_code_scanner/scan:
          event_before: << pipeline.git.base_revision >>
```

## Usage
**1. Get a Nightfall API key.**

The Nightfall DLP Orb is powered by the Nightfall DLP API. Learn more and request access to a free API key **[here](https://nightfall.ai/api/)**. Alternatively, you can email **[sales@nightfall.ai](mailto:sales@nightfall.ai)** to provision a free account.

**2. Set up config file to specify detectors.**

- Place a `.nightfalldlp/` directory within the root of your target repository, and inside it a `config.json` file in which you can configure your detectors (see `Detectors` section below for more information on Detectors)
- See `Additional Configuration` section for more advanced configuration options
- Below is a sample `.nightfalldlp/config.json` file:

```json
{
  "detectors": [
    "CREDIT_CARD_NUMBER",
    "PHONE_NUMBER",
    "API_KEY",
    "CRYPTOGRAPHIC_KEY",
    "RANDOMLY_GENERATED_TOKEN",
    "US_SOCIAL_SECURITY_NUMBER",
    "AMERICAN_BANKERS_CUSIP_ID",
    "US_BANK_ROUTING_MICR",
    "ICD9_CODE",
    "ICD10_CODE",
    "US_DRIVERS_LICENSE_NUMBER",
    "US_PASSPORT",
    "EMAIL_ADDRESS",
    "IP_ADDRESS"
  ]
}
```
**3. Set up a few environment variables.**     
These variables should be made available to the Nightfall DLP orb by adding them to `Environment Variables` under `Project Settings` on the CircleCI console. Instructions [here](https://circleci.com/docs/2.0/env-vars/#setting-an-environment-variable-in-a-project):

- `NIGHTFALL_API_KEY`
    - Get a free Nightfall DLP API Key by registering for an account with the [Nightfall API](https://nightfall.ai/api)

- `GITHUB_TOKEN` (Optional)
    - Create a [Github Personal Access Token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) with the `repo` scope
    - This token is used to authenticate to Github to write Comments/Annotations to your Pull Requests and Pushes
    - If not included, we will only log the finding messages to the CircleCI Workflow UI

- `EVENT_BEFORE`
    - Make sure to include `event_before` as an argument with the value `<< pipeline.git.base_revision >>` in your circle config as shown in the example above
    - This allows the orb to scan against the diff between the current commit sha and the most recent commit sha to trigger a workflow on push events

- `BASE_BRANCH` (Optional)
    - In the case that you want to scan your Pull Requests against a branch separate from `master`, include the `base_branch` argument with your branch's name in your circle config
    - This will allow you to scan the diff between your Pull Request and `base_branch`

**4. Allow Uncertified Orbs**
To use our orb, make sure to allow uncertified orbs in Organization Settings. Instructions are [here](https://circleci.com/docs/2.0/orbs-faq/#using-uncertified-orbs). At this time, CircleCI still requires allowing un-certified orbs to use orbs developed by CircleCI Partners.
## Supported GitHub Events
The Nightfall DLP CircleCI Orb can run in a Circle Workflow triggered by the following events:

- `PULL_REQUEST`
- `PUSH`

The Nightfall DLP CircleCI Orb does not currently support forked repositories due to potential permissioning issues that may occur.

## Detectors
Each detector represents a type of information you want to search for in your code scans. A few examples of detectors Nightfall supports includes:

- `API_KEY`: A freeform string used for user verification to access online program functions.
- `CREDIT_CARD_NUMBER`: A 12 to 19 digit number used for payments and other monetary transactions.
- `CRYPTOGRAPHIC_KEY`: A string of characters used by an encryption algorithm to generate seemingly random tokens.
- `RANDOMLY_GENERATED_TOKEN`: A pseudo-random string generated by an encryption algorithm. This detector is more general than the API_KEY detector.
- `US_SOCIAL_SECURITY_NUMBER`: A 9 digit numeric string often used as a unique identification number for United States citizens and residents.
- Many more...

**Find a full list of supported detectors in the Nightfall API Documentation, after creating your account (per the instructions above).**

## Additional Configuration

You can add additional fields to your config to ignore tokens and files from being flagged, as well as specify which files to exclusively scan.

### Token Exclusion

To ignore specific tokens from being flagged, you can add the `tokenExclusionList` field to your nightfalldlp config. The `tokenExclusionList` is a list of strings, and it mutes findings that match any of the given regex patterns.

Here's an example use case:

```tokenExclusionList: ["NF-gGpblN9cXW2ENcDBapUNaw3bPZMgcABs", "^127\\."]```

In the example above, findings with the API token `NF-gGpblN9cXW2ENcDBapUNaw3bPZMgcABs` as well as local IP addresses starting with `127.` would not be reported. For more information on how we match tokens, take a look at [regexp](https://golang.org/pkg/regexp/).

### File Inclusion & Exclusion

To omit files from being scanned, you can add the `fileExclusionList` field to your nightfalldlp config. In addition, to only scan specific files, add the `fileInclusionList` to the config.

Here's an example use case:
```
    fileExclusionList: ["*/tests/*"],
    fileInclusionList: ["*.go", "*.json"]
```
In the example, we are ignoring all file paths with a `tests` subdirectory, and only scanning on `go` and `json` files.
Note: we are using [gobwas/glob](https://github.com/gobwas/glob) to match file path patterns. Unlike the token regex matching, file paths must be completely matched by the given pattern. e.g. If `tests` is a subdirectory, it will not be matched by `tests/*`, which is only a partial match.

## [Nightfall DLP API](https://nightfall.ai/api)
With the Nightfall API, you can inspect & classify your data, wherever it lives. Programmatically get structured results from Nightfall's deep learning-based detectors for things like credit card numbers, API keys, and more. Scan data easily in your own third-party apps, internal apps, and data silos. Leverage these classifications in your own workflows - for example, saving them to a data warehouse or pushing them to a SIEM. Request access & learn more **[here](https://nightfall.ai/api/)**.

## Versioning
The Nightfall DLP CircleCI Orb issues releases using semantic versioning.

## Support
For help, please email us at **[support@nightfall.ai](mailto:support@nightfall.ai)**.
