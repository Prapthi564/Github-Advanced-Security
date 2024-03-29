# Module 3: Code Scanning

## Getting Started

If you followed `Module 0 - Setup and Automation`, you will have already enabled _GitHub Advanced Security_ at both the repository and the organization level. If you are starting this module without having taken these steps, please review that module to enable _GitHub Advanced Security_.

Navigate to the `ghas-bootcamp-python` repository to begin working through this module. This repository should have at-least 2 code scanning findings with the **Default** and the **Extended** setup in this repository (as of October 25, 2023).

## How Code Scanning performs Static Analysis differently

- What Code Scanning is doing under the hood with the CodeQL engine is as follows:

    Comprehensive Static Analysis differs from other solutions in that it needs access to the source code and plugs into the build process for compiled languages (or performs an approximate “compilation” for interpreted languages). This process leads to accurate data flow mapping and the ability to distinguish between remote and local sources, which in turn means fewer (but highly accurate and exploitable) findings.

- The fundamental difference between Code Scanning and other Static Analysis tools is that all of the information about the application is aggregated in a relational database that allows for tracing complete data flows across the entire application.

- For `compiled` languages, the CodeQL engine running under the hood of the Code Scanning process will hook into the compiler at build time. The CodeQL engine will then listen for the creation of data flows by the compiler, such as linkers and callbacks, and map those data flows as nodes in the database -- aptly called `DataFlow` nodes.
  - This process allows CodeQL to avoid false positive vulnerability findings from dead code that has no existing dataflows. This is a common problem with other Static Analysis tools that do not have access to the compiler and instead rely on pattern matching and other techniques to identify vulnerabilities.

- Once the data flow analysis is complete, an extraction of the code is then performed. Every variable, expression (combination or modification of variable(s)), method/function/class declaration, etc. is extracted as individual nodes in the database.

- CodeQL then performs analysis by querying the database for _Remote_ flow sources that lead to sinks (where data is stored or executed) in ways that are exploitable and are otherwise not sanitized as part of that data flow.

- For `interpreted` languages, like Javascript and Python, the CodeQL engine performs a depth-first, recursive extraction of the code where `DataFlow` nodes are created from things like `return` statements and passing variables from one function to another. We can gain a comprehensive view of the application and avoid flagging false positive vulnerabilities in code that is never called or executed.

## Default Setup

- In the `ghas-bootcamp-python` repository, go to **Settings** -> **Code security and analyis** -> Scroll down to **Code scanning** and click **Set up**. This will give you the option of _Default_ and _Advanced_ - for now click _Default_.

- The _Default_ window will give the option of which _Query suites_ you want to use.
- Leave the query suite on _Default_ and click the **Enable CodeQL** button. While that runs, explain the difference between the two query suites.
  - The _Default_ query suite (also known as the `code-scanning` query suite in the _Advanced_ setup) has a less than 10% False Positive rate from findings within the Open Source ecosystem. We focus very heavily on providing true positive findings that are remotely exploitable, and this suite is the most "dialed in" in terms of findings.
  - The _Extended_ query suite (also known as the `security-extended` query suite in the _Advanced_ setup) has a less than 30% False Positive rate from findings within the Open Source ecosystem. You will find a number of interesting queries get pulled into this suite, including _Memory Exploitation_ findings for C/C++ and other slightly more niche security vulnerabilities in other languages.

![default-setup](https://user-images.githubusercontent.com/22803099/236022918-3a0ab154-30f6-4514-bf96-165859f31d63.png)

- After you've clicked the **Enable CodeQL** button, go to the _Actions_ tab to confirm that the initial scan has kicked off. The scan should take a couple of minutes - but while that runs, we're going to turn on **Default** setup at the **Organization** level.

- Go to `ghas-bootcamp-DATE-HANDLE` and then **Settings** -> **Code security and analyis** -> scroll down to **Code scanning** and click **Enable all**. This will setup the _Default_ code scanning action with the _Default_ query suite on all eligible repositories (which is available for all CodeQL supported languages as of October 26, 2023).

## Advanced Setup

- Next, we're going to enable _Advanced Setup_ for one of our compiled language repositories by going back to the `ghas-bootcamp-java` repository, heading over to **Settings** -> **Code security and analysis** -> scroll down to **Code scanning** and clicking the `Set up` button and then clicking _Advanced_. Here, we are going to copy **Line 55** of the workflow file `# queries: security-extended,security-and-quality` and append this to a new line we create for **Line 50** which will read `queries: security-extended`. Commit these changes to your main branch and go to the **Actions** tab to confirm the CodeQL action is running.

![default-to-advanced](https://user-images.githubusercontent.com/22803099/236023025-12cee41d-5c93-404f-b3f2-595fe1149e49.png)

- The Action takes approximately 3 minutes to run and should produce one `Query built from user-controlled sources` (SQL Injection) finding. During this time, it's worth reiterating what is happening behind the scenes with the engine hooking the compiler to build dataflows. It's also worth reiterating the differences between the _default_, _security-extended_, and _security-and-quality_ query suites during this time.

- Take some time to review this finding by going to **Security** -> **Code scanning** and clicking on the `Query built from user-controlled sources` finding. Highlight the **Show paths** functionality as well as the **Show more** details section below on how we highlight good (and bad) coding practices that make for more secure software.

![show paths](https://user-images.githubusercontent.com/22803099/236023303-bb2c4113-4de8-4c17-905d-6fb7511269b3.png)

## Pull Request scans and Accurate Findings

- Next, we're going to enable _Advanced setup_ for one of our interpreted language repositories by going back to the `ghas-bootcamp-python` repository, heading over to **Settings** -> **Code security and analysis** -> scroll down to **Code scanning** and click the `...` and then click _Switched to advanced_. This will prompt us to turn-off the existing CodeQL workflow in order to avoid duplicating Actions runs.

- We are going to make similar updates to the `codeql.yml` file as we did in the `Advanced Setup` section by copying **Line 55** of the workflow file `# queries: security-extended,security-and-quality` and append this to a new line we create for **Line 50** which will read `queries: security-extended`.

![advanced-setup-codeql-yaml-changes](https://user-images.githubusercontent.com/22803099/236023514-cb7f0774-3c73-4408-b2bb-ba2a8995cca0.png)

- Open `server/routes.py` in the Python repository and scroll down to **Line 40**.
  - Ask the participants if there any vulnerabilities that stand out to them? (Hint: it has to do with SQL).

- Uncomment the Python code starting from **line 41**, commit these changes to a new branch, and open a **Pull request** into the **main** branch.
  - Because our _main_ branch isn't a `Protected Branch`, it may take a moment for the Action to trigger and the **Merge pull request** button will display _green_ until the Action kicks-off.

![pull-request-scan](https://user-images.githubusercontent.com/22803099/236024225-46d38699-ffff-4067-9fff-0e4c5a92e8b2.png)

- CodeQL does not flag this pull request with a _Query built from user-controlled sources_ finding. But why?
  - Go to the **Files chyanged** tab and review the code we added.
  - What we find on **Line 54** is a `val` assignment calling to `login.objects.raw` - which does not exist as a function in this project.
    - While other Static Analysis tools would likely have marked **Line 50** as a SQL Injection, the CodeQL analysis performed as part of Code Scanning correctly marks this as `Clear-text logging of sensitive information` finding. This is the power of CodeQL in action - accurately tracing dataflows and identifying security vulnerabilities in your code without all of the noise.

![clear-text-logging-finding](https://user-images.githubusercontent.com/22803099/236024335-4a1c08bd-8885-4780-b489-3fc6e42838b0.png)

- And with that, you've completed Module 3! If you're following this course in order, you can now move on to **Module 4 - Building a DevSecOps Program** 🎉