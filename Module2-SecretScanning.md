# Module 2: Secret Scanning

## Lab Objectives
- Task 1: Turn on secret scanning for the entire organization. Review results
- Task 2: Turn on push protection and try pushing a new secret (GitHub token) in a repository
- Task 3: Create a custom secret pattern and view the results 
- Task 4: Understanding the results 

## Getting Started

If you followed `Module 0 - Setup and Automation` you will have already enabled _GitHub Advanced Security_ at both the repository and the organization level. If you are starting this module without having taken these steps, below are the instructions for turning on these features at the organization level:

- Go to the Organization (`ghas-bc-HANDLE`) and then **Settings** -> **Code Security and Analysis** and click `Enable all` for _GitHub Advanced Security_. It may take several minutes for the repositories to all be flagged with GHAS enabled, so you may need to fill the time by discussing what Secret Scanning does before working through the module.

At the Organization level, `Enable all` for _Secret Scanning_. Also choose the checkboxes to opt in for new repos, validity checks, and non-provider patterns.

Once this is enabled, navigate to the `ghas-bootcamp-javascript` repository to begin working through this module. This repository should have at-least 4 secret scanning findings (as of Dec 12th, 2023).

![secret-scanning-findings](https://github.com/ghas-bootcamp-admin/training-material/assets/1760475/63ca66cf-c15f-4703-9553-6287a14724bd)

## Task 1: Turn on secret scanning for the entire organization. Review results

## What is Secret Scanning?
Before you share your screen, you'll want to start with a baseline understanding by first asking students what _they_ think Secret Scanning is. More often than not, students expect Secret Scanning to discover everything from AWS Credentials to hard-coded passwords like `Password123!`. Start by sharing what Secret Scanning can do out-of-the-box, and then mention _Custom Secret Patterns_ with plans to discuss it later in the module (time permitting).

  - As you walk through what Secret Scanning finds out-of-the-box, pull up the [Secret Scanning Patterns](https://docs.github.com/en/enterprise-cloud@latest/code-security/secret-scanning/secret-scanning-patterns) page to show which partner alerts we identify today. Make sure you are on the correct `Version` of the docs page when sharing this!

  - Next, show Secret Scanning in action by first reviewing the findings in `ghas-bootcamp-javascript`, followed by creating your own GitHub PAT and adding it to the code to show _Active Secrets_ detection.


1. To create a GitHub PAT token, go to your profile, and then select **Settings** -> **Developer settings** -> **Personal access tokens** -> **Tokens (classic)** and then clicking on **Generate new token** at the top and selecting **Generate new token (classic)**

![new personal acess token](images/PATtoken.png)

2. Give your secret a name **Secret scanning** and set the **Expiration** to _"Custom..."_ and select the next calendar day. By default, no permissions are granted so scroll to the bottom and click **Generate token**. Once you've generated the token, click the "Copy" icon to the right of the secret value, and return to the `ghas-bootcamp-javascript` repository.

3. Open _index.js_ file and click the pencil icon on the top-right of the code block to edit it and add ` var secret = "Your-Secret-Value"` to the code.

   **Note:** Replace "Your-Secret-Value" with the secret value copied in the previous step. 

4. Commit this to the default branch (Click on I'll fix this later if asked), and then **navigate to the Security -> Secret Scanning** section to show how this is an active secret. **Please note** that it may take a moment for this secret to be discovered, and the commit author will receive an email once it has been found (as long as you are not ignoring the repository in your watch settings).

5. Once the secret is located in **Security** -> **Secret Scanning**, it's time to discuss validity checks for "Active" secrets. As of March 20th, 2023 we are currently working through the legal process to get agreement from our existing Secret Scanning partners on using endpoints to test whether key material is active. We have prioritized Cloud platforms first (Azure, AWS, and Google) as part of this process.

6. Validity checks help development and security teams prioritize remediation of known active secrets by rotating those credentials and updating the codebase. For secrets found in commit history, the Field Security team recommends using [BFG Repo Cleaner](https://rtyley.github.io/bfg-repo-cleaner/) to remove secrets in code history. The only _real_ way to secure a secret is to rotate that credential and leverage a secrets management platform to access that secret in code.

![active-secret](https://user-images.githubusercontent.com/22803099/236021632-fd1e7325-0263-45a5-99cb-ebc0d16b84f6.png)

7. From here, go back to your profile, and then **Settings** -> **Developer settings** -> **Personal access tokens** -> **Tokens (classic)** and delete the secret you just pasted into the `ghas-bootcamp-javascript` repository. Once you've completed this step, go back to the **Security** -> **Secrets Scanning** section and review that the secret is now identified as a _"Possibly active secret"_.

![possibly-active-secret](https://user-images.githubusercontent.com/22803099/236021678-6d1e32a1-846a-4462-980b-0a4a5714e7df.png)

## Task 2: Turn on push protection and try pushing a new secret (GitHub token) in a repository

1. Go to the `ghas-bootcamp-javascript` repository and enable Push Protection. Go to **Settings** -> **Code Security and Analysis** -> scroll down to **Push protection** and click **Enable**.

2. After this is enabled, we are going to go back through the process of creating a secret and pasting it into the codebase. Once again go to your profile, and then **Settings** -> **Developer settings** -> **Personal access tokens** -> **Tokens (classic)** and then clicking on **Generate new token** at the top and selecting **Generate new token (classic)**.

3. From here, give your secret a name **secret2** and set the **Expiration** to _"Custom..."_ and select the next calendar day. By default, no permissions are granted so it is safe to scroll to the bottom and click **Generate token**.

4. Once you've generated the token, click the "Copy" icon to the right of the secret value, and return to the `ghas-bootcamp-javascript` repository. Open up _index.js_, click the pencil icon on the top-right of the code block and add `var secret2 = "Your-Secret-Value"` to the code. Commit the changes with the default options to the to attempt to push the code. This will cause a **secret scanning** pop-up to appear, stopping you from committing your secret to the codebase.

  ![push-protection](./images/sscanningpop.png)

  - This is a great opportunity to pause and discuss **WebHooks** - specifically how Security findings generate event driven outputs, which can be captured via Security Incident and Event Management (SIEM) integrations for monitoring and alerting. The important thing to mention here is that we take a "trust but verify" approach.

  - It's worth mentioning that we have all heard about companies experiencing outages when hard-coded secrets are removed, and in such situations business leaders are going to expect developers to bring the service back online. We allow for push protection bypasses to take place for exactly this reason. If this happens on a repository of consequence for a company - security teams are going to want to know about it, which is where **WebHooks** can make a difference.

5. Finally, to show off how much importance we place on catching real secrets over modified / false positive secrets, we will go back through the process of creating a new personal acess token. Once again go to your profile, and then **Settings** -> **Developer settings** -> **Personal access tokens** -> **Tokens (classic)** and then clicking on **Generate new token** at the top and selecting **Generate new token (classic)**.

6. Once again, give your secret a name **secret3** and set the **Expiration** to _"Custom..."_ and select the next calendar day. By default, no permissions are granted so it is safe to scroll to the bottom and click **Generate token**.

7. Once you've generated the token, click the "Copy" icon to the right of the secret value, and return to the `ghas-bootcamp-javascript` repository. Open up _index.js_, click the pencil icon on the top-right of the code block and add `var secret3 = "Your-Secret-Value"` to the code. **BEFORE YOU COMMIT YOUR CODE** go ahead and add some random letters and numbers to the end of the GitHub Personal Access Token you've added.

8. Before you've committed your changes, ask the students what they expect to happen. Some will probably guess that you should see a push protection pop-up. Other's will have clued into the importance of only flagging on real secrets, and will say that you should not see a pop-up. In the end, this will _NOT_ cause a Push Protection pop-up to appear. It's important to once-again reiterate that we focus on Push Protecting against secrets that we are highly confident are real, and that match the patterns expected from the algorithms are partners use to generate their credential material.

![false-positive-secret-push-protection-test](https://user-images.githubusercontent.com/22803099/236021931-aee23d97-03e3-40c7-a129-bd868c25226e.png)

## Task 3: Create a custom secret pattern and view the results 

- Within the `ghas-bootcamp-javascript` repo there is a secret disclosed on line 27 of the file `index.js`. To discover secrets like this, we will go into this repository's **Settings** -> **Code Security and Analysis** -> **Secret Scanning** section and then click on the button **New pattern**.

![new-pattern-repository](https://user-images.githubusercontent.com/22803099/236021992-55d6faf3-5986-487d-93e7-b838b73b2f3e.png)

- When creating a new pattern, first supply a name and enter a test string for the pattern you're going to write. Because writing regular expression patterns can be challenging, we recommend using something like _GitHub Copilot_ or [Regex101.com](https://regex101.com/) to help with this process. You can also refer to the Field Security team's open source [custom-pattern-secrets](https://github.com/advanced-security/secret-scanning-custom-patterns) repository which contains a number of useful examples.

- Using the [Generic Secrets / Passwords](https://github.com/advanced-security/secret-scanning-custom-patterns/tree/main/generic) from the Field Security team's repository, we grab the following pattern format:

  - ``[a-zA-Z0-9!.,$%&*+?^_`{|}()[\]\\/~-][a-zA-Z0-9\t !.,$%&*+?^_`{|}()[\]\\/~-]*``

- You will also want to add a couple of values to the **Before secret** and **After secret** values. The following can be used for the **Before Secret**:

  - `(?:\A|[^a-zA-Z0-9])(?i)(?:api|jwt|mysql)?[_.-]?(?:[Pp]ass?(?:wo?r?d|code|phrase)|[Pp]wd|secret)[\t ]*(={1,3}|:)[\t ]*(?:["']|b["'])?`

- The **After secret** that works in our example is as follows:

  - `(\z|[\r\n'"])`

- Click **Save and dry run** to discover two values of concern - the first being the GitHub Personal Access Token we corrupted earlier, and the second being the `Password123!` value mentioned earlier.

- This example captures everything we expect to see in a single repository - which can hold up to 100 patterns. Let's go ahead and apply this to the organization level by going to the `ghas-bc-HANDLE` and clicking **Settings** -> **Code security and analysis** -> **Secret scanning** and click the green **New pattern** button. Please note that the Organization can hold up to 500 custom patterns.

- Go ahead and apply the same pattern we used earlier at the repository level and click **Save and dry run**. You'll get a pop-up asking where to apply this secret; click the top radial that indicates "All repositories in this organization". The UI will then indicate that the dry run is queued - which is usually a great time to take a 10 or 15 minute break.

- Wrap up this section by refreshing the page and doing a cursory review of the findings. You'll notice there are some false positives identified with the custom pattern we've written - if you're doing this in a two day format, encourage the students to spend some time refining the pattern further using the **Add requirement** button and see what they can come up with.

- Even if you're doing this in a one-day format, it's worth highlighting the value of performing the **Dry run** before publishing the pattern to avoid notifying developers of false positives.

![some-false-positives](https://user-images.githubusercontent.com/22803099/236022561-fc998b79-c641-472e-bc8b-64d98b8b4c34.png)

- And with that, you've completed Module 2! If you're following this course in order, you can now move on to **Module 3 - Code Scanning** 🎉