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
  nightfall_code_scanner: nightfall/nightfall_code_scanner@v2.0.0

workflows:
  test_scanner:
    jobs:
      - nightfall_code_scanner/scan:
          event_before: << pipeline.git.base_revision >>
```

## Usage
**1. Get a Nightfall API key.**

The Nightfall DLP Orb is powered by the Nightfall DLP API. Learn more and request access to a free API key **[here](https://nightfall.ai/api/)**. Alternatively, you can email **[sales@nightfall.ai](mailto:sales@nightfall.ai)** to provision a free account.

**2. Set up config file to specify condition set.**

- Place a `.nightfalldlp/` directory within the root of your target repository, and inside it a `config.json` file in which you can configure your condition set (see the `Conditions` section below for more information)
- See `Additional Configuration` section for more advanced configuration options

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

## Nightfalldlp Config File

The .nightfalldlp/config.json file is used as a centralized config file to control what conditions/detectors to scan with and what content you want to scan for in your pull requests. It includes the following adjustable fields to fit your needs.

### ConditionSetUUID

A condition set uuid is a unique identifier of a condition set, which can be created via [app.nightfall.ai](app.nightfall.ai).
Once defined, you can simply input the uuid in your config file, e.g.

```json
{ "conditionSetUUID": "A0BA0D76-78B4-4E10-B653-32996060316B" }
```

Note: by default, if both conditionSetUUID and conditions are specified, only the conditionSetUUID will be used.

### Conditions

Conditions are a list of conditions specified inline. Each condition contains a nested detector object as well as two additional parameters: minNumFindings and minConfidence.

```json
{
  "conditions": [
    {
      "minNumFindings": 1,
      "minConfidence": "POSSIBLE",
      "detector": {}
    }
  ]
}
```

minNumFindings specifies the minimal number of findings required to return for one request, e.g. if you set minNumFindings to be 2, and only 1 finding within the request payload related to that detector, that finding then will be filtered out from the response.

minConfidence specifies the minimal threshold to trigger a potential finding to be captured. We have five levels of confidence from least sensitive to most sensitive:

- VERY_LIKELY
- LIKELY
- POSSIBLE
- UNLIKELY
- VERY_UNLIKELY

### Detector

A detector is either a prebuilt Nightfall detector or custom regex or wordlist detector that you can create. This is specified by the detectorType field.

- nightfall prebuilt detector

  ```json
  {
    "detector": {
      "detectorType": "NIGHTFALL_DETECTOR",
      "nightfallDetector": "API_KEY",
      "displayName": "apiKeyDetector"
    }
  }
  ```

  Within detector struct

  - First specify detectorType as NIGHTFALL_DETECTOR
  - Pick the nightfall detector you want from the list
    - API_KEY
    - CRYPTOGRAPHIC_KEY
    - RANDOMLY_GENERATED_TOKEN
    - CREDIT_CARD_NUMBER
    - US_SOCIAL_SECURITY_NUMBER
    - AMERICAN_BANKERS_CUSIP_ID
    - US_BANK_ROUTING_MICR
    - ICD9_CODE
    - ICD10_CODE
    - US_DRIVERS_LICENSE_NUMBER
    - US_PASSPORT
    - PHONE_NUMBER
    - IP_ADDRESS
    - EMAIL_ADDRESS
  - Put a display name for your detector, as this will be attached on your findings

- customized regex

  We also support customized regex as a detector, which are defined as followed:

  ```json
  {
    "detector": {
      "detectorType": "REGEX",
      "regex": {
        "pattern": "coupon:\\d{4,}",
        "isCaseSensitive": false
      },
      "displayName": "simpleRegexCouponDetector"
    }
  }
  ```

- word list

  Word list detectors look for words you specify in its list. Example below:

  ```json
  {
    "detector": {
      "detectorType": "WORD_LIST",
      "wordList": {
        "values": ["key", "credential"],
        "isCaseSensitive": false
      },
      "displayName": "simpleWordListKeyDetector"
    }
  }
  ```

- [Extra Parameters Within Detector]

  Aside from specifying which detector to use for a condition, you can also specify some additional rules to attach. They are contextRules and exclusionRules.

  - contextRules
    A context rule evaluates the surrounding context(pre/post chars) of a finding and adjusts the finding's confidence if the input context rule pattern exists.

    Example:

    ```json
    {
      "detector": {
        // ...... other detector fields
        "contextRules": [
          {
            "regex": {
              "pattern": "my cc",
              "isCaseSensitive": true
            },
            "proximity": {
              "windowBefore": 30,
              "windowAfter": 30
            },
            "confidenceAdjustment": {
              "fixedConfidence": "VERY_LIKELY"
            }
          }
        ]
      }
    }
    ```

    - regex field specifies a regex to trigger
    - proximity is defined as the number pre|post chars surrounding the finding to conduct the search
    - confidenceAdjustment is the confidence level to adjust the finding to upon existence of the input context

    As an example, say we have the following line of text in a file `my cc number: 4242-4242-4242-4242`, and `4242-4242-4242-4242` is detected as a credit card number with confidence of POSSIBLE. If we had the context rule above, the confidence level of this finding will be bumped up to `VERY_LIKELY` because the characters preceding the finding, `my cc`, match the regex.

  - exclusionRules
    Exclusion rules on individual conditions are used to mute findings related to that condition's detector.

    Example:

    ```json
    {
      "detector": {
        // ...... other detector fields
        "exclusionRules": [
          {
            "matchType": "PARTIAL",
            "exclusionType": "REGEX",
            // specify one of these values based on the type specified above
            "regex": {
              "pattern": "4242-4242-\\d{4}-\\d{4}",
              "isCaseSensitive": true
            },
            "wordList": {
              "values": ["4242"],
              "isCaseSensitive": false
            }
          }
        ]
      }
    }
    ```

    - exclusionType specifies either a REGEX or WORD_LIST
    - regex field specifies a regex to trigger, if you choose to use REGEX type
    - matchType could be either PARTIAL or FULL, To be a full match, the entire finding must match the regex pattern or word exactly, whereas findings containing more than just the regex pattern or word are considered partial matches. Example: suppose we have a finding of "4242-4242" with exclusion regex of 4242. If you use PARTIAL, this finding will be deactivated, while FULL not, since the regex only matches partial of the finding

## Additional Configuration

You can add additional fields to your Nightfall config to ignore tokens and files from being flagged, as well as specify which files to exclusively scan.

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
