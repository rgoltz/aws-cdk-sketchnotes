# AWS CDK - rogo @ Fedora

## Prerequisites

### GitHub

* Update the fork with the current main branch from aws-cdk.
* Pull the current state to local notebook in VS Code.

### Local Environment Setup

Required tools and dependencies (I need to update this ...):

- **Node.js** (LTS version)
- **yarn** (package manager)
- **AWS CLI** (for account access)
- **GitHub CLI** (version control)
- ...


#### Development + System Requirements
- I've added 16 GB as SWAP-file on my local notebook ...
- I'm running all steps in a local Shell - Hence, I've closed closed VS Code, browsers, and other memory-intensive applications during builds to prevent out-of-memory errors.


#### Shell Environment Variables

Set Node.js memory limits:

```bash
export NODE_OPTIONS="--max-old-space-size=8192"
```

#### AWS Account Setup

"Create" (or use) an AWS account and configure credentials:

```bash
export AWS_ACCESS_KEY_ID=G..
export AWS_SECRET_ACCESS_KEY=..AY
export AWS_DEFAULT_REGION=us-west-2
```

Verify AWS connection:

```bash
aws sts get-caller-identity
```

#### CDK Bootstrap

For new or temporary accounts, bootstrap CDK (replace with the account ID):

```bash
cdk bootstrap 1234567890/us-west-2
```

## Development Workflow

### 1. Install Dependencies

Directory: `~/git/github/aws-cdk/` (root)
```bash
yarn install
```

### 2. Build Integration Test Framework

Directory: `~/git/github/aws-cdk/` (root)
```bash
npx lerna run build --scope=@aws-cdk-testing/framework-integ
```

**Important**: The integ-runner works with compiled JavaScript files (.js), not TypeScript files (.ts)!

### 3. Code Changes

Make changes in the appropriate package path: `packages/aws-cdk-lib/<aws-component>`

Examples:
* `packages/aws-cdk-lib/aws-logs/lib/data-protection-policy.ts`
* `packages/aws-cdk-lib/aws-elasticloadbalancingv2/lib/alb/application-load-balancer.ts`

### 4. Unit Tests

Unit tests are located in corresponding test directories:

* TypeScript file directly under `lib/` → `test/`
* TypeScript file in subfolder, e.g., `lib/alb/` → `test/alb/`

#### Running Unit Tests

Directory: `~/git/github/aws-cdk/packages/aws-cdk-lib/`

Command:

```bash
# ensure that you are on the correct git branch - could be checked via: git branch --show current
npx jest aws-logs/test/loggroup.test.ts --testNamePattern="DateOfBirth"
```

<details>

<summary> [expand me *click*] Example for Unit-Test in Shell</summary>

```
robert@fedora:~/git/github/aws-cdk/packages/aws-cdk-lib$ npx jest aws-logs/test/loggroup.test.ts --testNamePattern="DateOfBirth"
 PASS  aws-logs/test/loggroup.test.ts
  ○ skipped set field index policy with four fields indexed
  ○ skipped set more than 20 field indexes in a field index policy
  ○ skipped encrypting log group with referenced alias
  ○ skipped create a Add Key transformer against a log group
  log group
    ✓ set data protection policy with DateOfBirth identifier (91 ms)
    ○ skipped set kms key when provided
    ○ skipped fixed retention
    ○ skipped default retention
    ○ skipped infinite retention/dont delete log group by default
    ○ skipped infinite retention via legacy method
    ○ skipped unresolved retention
    ○ skipped with INFREQUENT_ACCESS log group class
    ○ skipped with STANDARD log group class
    ○ skipped with default log group class
    ○ skipped will delete log group if asked to
    ○ skipped import from ARN, same region
    ○ skipped import from ARN, different region
    ○ skipped import from name
    ○ skipped extractMetric
    ○ skipped extractMetric allows passing in namespaces with "/"
    ○ skipped grant write
    ○ skipped grant read
    ○ skipped grant to service principal
    ○ skipped when added to log groups, IAM users are converted into account IDs in the resource policy
    ○ skipped log groups accept the AnyPrincipal policy
    ○ skipped imported values are treated as if they are ARNs and converted to account IDs via CFN pseudo parameters
    ○ skipped correctly returns physical name of the log group
    ○ skipped set data protection policy with custom name and description and no audit destinations
    ○ skipped set data protection policy string-based data identifier
    ○ skipped set data protection policy with audit destinations
    ○ skipped set data protection policy with custom data identifier
    ○ skipped set data protection policy with mix of managed and custom data identifiers
    loggroups imported by name have stream wildcard appended to grant ARN
      ○ skipped case 1
      ○ skipped case 2
    loggroups imported by ARN have stream wildcard appended to grant ARN
      ○ skipped case 1
      ○ skipped case 2
  subscription filter
    ○ skipped add subscription filter with custom name


=============================== Coverage summary ===============================
Statements   : 35.41% ( 4901/13839 )
Branches     : 14.43% ( 763/5286 )
Functions    : 18.21% ( 554/3041 )
Lines        : 36.15% ( 4747/13131 )
================================================================================
Jest: "global" coverage threshold for statements (55%) not met: 35.41%
Jest: "global" coverage threshold for branches (35%) not met: 14.43%
Test Suites: 1 passed, 1 total
Tests:       36 skipped, 1 passed, 37 total
Snapshots:   0 total
Time:        4.383 s
Ran all test suites matching /aws-logs\/test\/loggroup.test.ts/i with tests matching "DateOfBirth".
```

</details>
<br>

### 5. Rebuild After Changes

After making changes, rebuild the integration test framework:

```bash
npx lerna run build --scope=@aws-cdk-testing/framework-integ
```
You can also try (not tested here):
```bash
npx lerna run build --scope=@aws-cdk
```



### 6. Commit Your Changes

Now you can commit your code changes, unit tests, and integration tests:

Example PR: https://github.com/aws/aws-cdk/pull/36558
* Code changes in `data-protection-policy.ts`
* Unit test in `loggroup.test.ts`
* Integration test in `integ.log-group.ts`

### 7. Integration Tests

Now the AWS account comes into play. Integration tests need updates, and the integ-runner handles this.

#### Update Integration Test Snapshots

Directory: `~/git/github/aws-cdk/packages/@aws-cdk-testing/framework-integ/`

Command to update snapshots and CloudFormation templates:

```bash
npx integ-runner --parallel-regions us-west-2 --update-on-failed test/aws-logs/test/integ.log-group.js
```


<details>

<summary>[expand me *click*] Example Integration Test Output (Deploy to Account as CloudFormation Stack and Template Update)</summary>

```
Verifying integration test snapshots...

  CHANGED    aws-logs/test/integ.log-group 1.901s
      Resources
      [~] AWS::Logs::LogGroup LogGroupLambdaAC756C5B
       └─ [~] DataProtectionPolicy
           └─ [~] .Statement:
               └─ @@ -22,6 +22,18 @@
                  [ ]       {
                  [ ]         "Ref": "AWS::Partition"
                  [ ]       },
                  [+]       ":dataprotection::aws:data-identifier/DateOfBirth"
                  [+]     ]
                  [+]   ]
                  [+] },
                  [+] {
                  [+]   "Fn::Join": [
                  [+]     "",
                  [+]     [
                  [+]       "arn:",
                  [+]       {
                  [+]         "Ref": "AWS::Partition"
                  [+]       },
                  [ ]       ":dataprotection::aws:data-identifier/EmailAddress"
                  [ ]     ]
                  [ ]   ]
                  @@ -68,6 +80,18 @@
                  [ ]       {
                  [ ]         "Ref": "AWS::Partition"
                  [ ]       },
                  [+]       ":dataprotection::aws:data-identifier/DateOfBirth"
                  [+]     ]
                  [+]   ]
                  [+] },
                  [+] {
                  [+]   "Fn::Join": [
                  [+]     "",
                  [+]     [
                  [+]       "arn:",
                  [+]       {
                  [+]         "Ref": "AWS::Partition"
                  [+]       },
                  [ ]       ":dataprotection::aws:data-identifier/EmailAddress"
                  [ ]     ]
                  [ ]   ]
      
      

Snapshot Results: 

Tests:    1 failed, 1 total
Failed: /home/robert/git/github/aws-cdk/packages/@aws-cdk-testing/framework-integ/test/aws-logs/test/integ.log-group.js

Running integration tests for failed tests...

Running in parallel across regions: us-west-2
Running test /home/robert/git/github/aws-cdk/packages/@aws-cdk-testing/framework-integ/test/aws-logs/test/integ.log-group.js in us-west-2
  SUCCESS    aws-logs/test/integ.log-group-LogGroupInteg/DefaultTest 104.384s
      

Test Results: 

Tests:    1 passed, 1 total
```

</details>
<br>

### 8. Additional Integration Test Commands

```bash
# Run the Linter locally
npx lerna run lint --scope=aws-cdk-lib

# Clean up test resources
npx integ-runner --clean
```
