# 🔑 Sops Secrets Construct Library

[![npm](https://img.shields.io/npm/v/cdk-sops-secrets.svg)](https://www.npmjs.com/package/cdk-sops-secrets)&nbsp;&nbsp;
[![Known Vulnerabilities](https://snyk.io/test/github/markussiebert/cdk-sops-secrets/badge.svg)](https://snyk.io/test/github/markussiebert/cdk-sops-secrets)&nbsp;&nbsp;
[![release](https://github.com/markussiebert/cdk-sops-secrets/actions/workflows/release.yml/badge.svg)](https://github.com/markussiebert/cdk-sops-secrets/actions/workflows/release.yml)

This construct library provides a replacement for CDK SecretsManager Secrets, with extended functionality for mozilla/sops.

Using this library it is possible to populate Secrets with values from a mozilla/sops file without additional scripts and steps in the CI stage. Thereby transformations like JSON conversion of YAML files and transformation into a flat, JSONPath like structure can be done.

Secrets filled in this way can be used immediately within the CloudFormation stack and dynamic references. This construct should handle all dependencies, if you use the ```secretValueFromJson``` call. 

This way, secrets can be securely stored in git repositories and easily synchronized into AWS SecretsManager Secrets.

## Getting started

1. Create a mozilla sops secrets file (with kms) and place it somewhere in your git repository
2. Create a secret with the SopsSecret construct
   ```ts
    const secret = new SopsSecret(stack, 'SopsComplexSecretJSON', {
        sopsFilePath: 'secets/sopsfile-encrypted.json',
    });
   ```
3. Optional: Access the secret via dynamic references
   ```ts
   secret.secretValueFromJson('json.path.dotted.notation.accessor[0]').toString(),
   ```


## Motivation

This project was created to solve a recurring problem of syncing mozilla/sops secrets into AWS SecretsManager in a convenient, secure way.
 
Other than that, or perhaps more importantly, my goal was to learn new things:
- Write a golang lambda
- Writing unit tests incl. mocks in golang
- Reproducible builds of golang binaries (byte-by-byte identical)
- Build reproducible zips (byte-by-byte identical)
- Release an npm package
- Setting up projects with projen
- CI/CD with github actions
- CDK unit and integration tests

## Other Tools like this

The problem this Construct addresses is so good, already two other implementations exist:

* [isotoma/sops-secretsmanager-cdk](https://github.com/isotoma/sops-secretsmanager-cdk): Does nearly the same. Uses CustomResource, wraps the sops cli, does not support flatten. Found it after I published my solution to npm :-/ 
* [taimos/secretsmanager-versioning](https://github.com/taimos/secretsmanager-versioning): Different approach on the same problem. This is a cli tool with very nice integration into cdk and also handles git versioning information.

## Advanced Configuration

There are more options to configure the Construct which will be explained in the further chapters

### Access the Lambda Role

You can access the Lambda Role by creating your own Provider

```typescript
    const myExtraKmsKey = Key.fromKeyArn(this, 'MyExtraKmsKey', 'YourKeyArn');
    const provider = new SopsSyncProvider(this, 'CustomSopsSyncProvider')
    myExtraKmsKey.grantDecrypt(provider.role!)
    const secret = new SopsSecret(this, 'SopsComplexSecretJSON', {
        sopsProvider: provider,
        secretName: 'myCoolSecret',
        description: "its extra cool",
        sopsFilePath: 'secrets/sopsfile-encrypted.json',
    });
```
