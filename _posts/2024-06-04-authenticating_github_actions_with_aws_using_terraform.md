---
title: Authenticating GitHub Actions with AWS using Terraform
image: /assets/images/rohit-tandon-nepal-mountains.jpg
---

So you want to work with AWS from your GitHub Actions workflow? This article guides you through authorizing your GitHub Actions workflow with AWS and helps you to understand the underlying mechanisms.

## Prerequisites

- AWS Account.
- Some knowledge on AWS IAM (Identity Access Management) roles and policies.
- Terraform 1.8.4.
- A Terraform configuration linked to your AWS account ([Tutorial](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-build)).

## OpenID Connect

OpenID Connect (OIDC) is the gold standard for authentication. It’s the simplest way to securely verify the identity of our GitHub Actions workflow with AWS. There are some great in-depth articles out there that can explain OIDC better than me like [this one](https://openid.net/developers/how-connect-works/) from the OpenID Foundation and [this one](https://developer.okta.com/blog/2019/10/21/illustrated-guide-to-oauth-and-oidc) from Okta, but I will briefly explain OIDC for the purpose of this article.

OIDC runs on the OAuth2 authorization framework. OAuth2 doesn’t provide a method for verifying the identity of a user or entity, rather it grants access to resources (e.g., An admin-only page, a database or a set of images). Using the authorization capabilities of OAuth2, OIDC securely verifies the identity of a user or entity. OIDC grants a login “session” to the client so the client can use a single identity to request multiple resources. Using GitHub’s OIDC Identity Provider (IdP) server, we can grant a login session to our GitHub Action workflow from AWS so that we can work with our AWS account from GitHub Actions.

## Terraforming the OIDC Configuration

When Terraform creates infrastructure in our AWS account, it assumes a role of an IAM user (which we should have set up when creating a Terraform configuration linked to our AWS account). Sometimes we need some information from this role (like the account ID) to complete certain actions. Since we don’t want to store these things in plain text, AWS has created the `aws_caller_identity` data source.

```javascript
  data "aws_caller_identity" "current" {}
```

We also need to define the IdP who is going to be giving out the identity of our GitHub Action to AWS. Similar to the `aws_caller_identity` data source, we need to define the location of this IdP in Terraform so that we can access information about the IdP later.

```javascript
data "aws_iam_openid_connect_provider" "github_actions" {
  url = "https://token.actions.githubusercontent.com"
}
```

Our GitHub Action identity must be registered in AWS in order for trust to be established between the two parties. To do this, we must create an IAM policy which uses information from both parties. Most of the information on the IAM policy principals and conditions for authorizing GitHub with AWS is going to be specific to GitHub and as such can be found in [this article](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services) by GitHub. If you are having trouble establishing an authorized connection between GitHub and AWS, I would start with that article since there are a few nuances depending on our environment. The following is the IAM policy document that works for me.

```javascript
data "aws_iam_policy_document" "github_attachment_policy" {
  version = "2012-10-17"
  statement {
    sid     = ""
    effect  = "Allow"
    actions = ["sts:AssumeRoleWithWebIdentity"]

    principals {
      type        = "Federated"
      identifiers = ["arn:aws:iam::${data.aws_caller_identity.current.account_id}:oidc-provider/token.actions.githubusercontent.com"]
    }

    condition {
      test     = "StringEquals"
      variable = "token.actions.githubusercontent.com:sub"
      values   = ["repo:${var.GITHUB_ORGANIZATION}/${var.GITHUB_REPO}:ref:refs/heads/${var.GIT_BRANCH}"]
    }

    condition {
      test     = "StringEquals"
      variable = "token.actions.githubusercontent.com:aud"
      values   = data.aws_iam_openid_connect_provider.github_actions.client_id_list
    }
  }
}
```

Take note how we are using the the `aws_caller_identity` and `aws_iam_openid_connect_provider` data sources from earlier.

<aside>
💡 <code>${var.VARIABLE}</code> is a Terraform variable. Learn how  to set up Terraform variables [here](https://developer.hashicorp.com/terraform/language/values/variables).
</aside>

<aside>
💡 <code>${var.GIT_BRANCH}</code> safeguards against unauthorized infrastructure changes. I set the branch in my configuration and only certain people can approve changes to it. This, in addition to AWS IAM role's least privilege, further secures the GitHub action.
</aside>

Next we specify the AWS IAM role that the GitHub Action is going to assume, and attach the policy we created to this role. I’ve included the `${var.GITHUB_REPO}` and `${var.GIT_BRANCH}` in the name of this role for easy identification (in case we have multiple branches or roles).

```javascript
resource "aws_iam_role" "github_pipeline" {
  name                 = "github_${var.GITHUB_REPO}_${var.GIT_BRANCH}_branch"
  description          = "Rule used by the github actions pipeline in the ${var.GITHUB_REPO} repository on the ${var.GIT_BRANCH} branch."
  max_session_duration = 3600 # 1 hour

  assume_role_policy = data.aws_iam_policy_document.github_attachment_policy.json
}
```

## Authorizing in GitHub Actions workflow

Finally we need to assume the role we created inside a GitHub Action workflow so that it can make changes in AWS. Make sure this workflow is running on the branch we specified in `${var.GIT_BRANCH}` earlier.

```yaml
name: Authenticate with AWS

on:
  push:
    branches:
      - 'branch_name'

permissions:
  id-token: write
jobs:
  authenticate-with-aws:
    steps:
      - name: Authenticate with AWS
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-region: ca-central-1
          role-to-assume: "arn:aws:iam::927869390122:role/github_${{ github.event.repository.name }}_${{ github.ref_name }}_branch"
```

The important things to note here are:

- Adding the `id-token: write` to the workflow so that the identity token can be fetched from GitHub’s IDP server and sent to AWS to authorize the workflow.
- Adding the `role-to-assume` as the role we created earlier in the “Authenticate with AWS” step that we created earlier using Terraform.

## Next Steps

To make any changes to AWS, create a new IAM policy with only the minimal amount of permissions. Then, connect this policy to the `github_pipeline` role we created in this article.

## Future Improvements and Considerations

Have any comments or improvements on the above article? Let me know by submitting a Pull Request on [this website’s GitHub repository](https://github.com/ryanshepps/ryansheppard.tech).