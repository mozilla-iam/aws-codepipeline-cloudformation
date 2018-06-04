# aws-codepipeline-cloudformation

### Summary

This CloudFormation stack will automate the creation of a CI/CD pipeline with
CodePipeline, CodeBuild and ECR. We intend on reusing this pattern to ensure
that we deploy secure and stable container images to our Kubernetes clusters.

We can use this pipeline to enforce certain actions, like building the Docker
container and scanning it with [Clair](https://github.com/coreos/clair) to check
for known security vulnerabilities. If we want to, deployments can be prevented
if a security vulnerability is discovered in the container image.

In addition to the benefits above, I expect to see us address some common
security issues by using this CloudFormation stack to create all of our
application pipelines. Doing this through code ensures that we repeat best
practices when creating resources in AWS. We can promote the good behavior like:

- Environment isolation: The build pipeline for the SSO dashboard can be
  completely isolated from the Mozillians build pipeline
- Granular permissions: If we create pipelines manually, we risk issues like
  forgetting to lock down IAM permissions. We can avoid this by doing it right
once and repeating that pattern
- Patching: Security vulnerabilities and be found with Clair and additional
  security checks can be put in place in other stages in the pipeline and then
replicated to all ouf our services

### Required Cloudformation stack parameters

```
  ProjectName:
    Description: The name for the project associated with this pipeline (used to namespace resources)
  GitHubOwner:
    Description: The owner or organization for the GitHub project
  GitHubRepo:
    Description: The GitHub repository name
  GitHubRepoBranch:
    Description: The branch for the GitHub repository
  GitHubOAuthToken:
    Description: The OAuth token to access GitHub
  Email:
    Description: The email address where CodePipeline sends pipeline notifications
```

### Architectural overview

This CloudFormation stack will take several parameters and build isolated
resources which are scoped to a single project or application. Today, it will
build the following:

- CodePipeline
  - A Source phase which polls the GitHub repository for changes
  - A build phase (a triggered CodeBuild job)
- CodeBuild
  - Build the Docker container image from the source repository
  - Run Clair against this container image
  - Note: both of these actions come from the `buildspec.yml` in the source
    repository
- ECR
  - Create a container registry for this specific application
- An SNS topic associated with CodePipeline for build notifications
- IAM configuration to allow access to required resources

### Known issues

The project is still new and there are some issues that I want to address
immediately.

- Find an alternate method of triggering the CodePipeline source so we do not
  have to manage a GitHub OAuthToken
- Modify this CloudFormation to transform it into a valid StackSet
- Complete the work to integrate Clair and consider providing a Clair
  `buildspec.yml` via CloudFormation rather than relying on one in the
application's source repository
- Revise the IAM configuration to grant the lowest level of access required for
  each resource
- Integrate testing and deployment phases into each CodePipeline
- Switch from an email SNS topic to something that can provide Slack notifications

