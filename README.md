# terratest_guideline

[![Build Status](https://travis-ci.org/foreverXZC/terratest_guideline.svg?branch=master)](https://travis-ci.org/foreverXZC/terratest_guideline)

## Terratest User Guideline

### About Terratest

#### Why Terratest

Terratest is a Go library that makes it easier to write automated tests for your infrastructure code. It provides a variety of helper functions and patterns for common infrastructure testing tasks, such as making HTTP requests and SSH to a particular virtual machine. Using terratest, we are able to automatically test a terraform module and see whether the infrastructure code runs properly.

#### When Terratest

When we would like to create new modules or update a new version of an existing module, we may want to test whether this infrastructure code is correct, especially before release. As a result, terratest plunges in and enables us to do so.

#### Compared to Kitchen Terraform

Kitchen Terraform is also a nice testing tool. It is not only responsible for creating and destroying a resource, but also checking properties in `tfstate` file. By contrast, when we would like to answer whether the infrastructure actually works, terratest is a better choice. We are able to make HTTP request to a server, or SSH to a virtual machine with help of terratest, and thus we can validate if our resources operate properly.

#### Advantages

1. Validation: Users can design how to validate whether the infrastructure code works instead of just checking specific files.

1. Simple file structure: Code organization is quite clear because users only need to design test case and test code.

1. Debug: Terratest is written in Golang, which is a mature compiled language which we can debug more conveniently than Ruby.

1. Lightweight: Users do not need to provide lots of configuration files and the project only depends on Golang.

1. Extension: It is not difficult to extend more functions on top of Terratest, such as Azure specific features.

1. Freedom: Users are able to write any kinds of test code they like, even if they are not related to Terratest itself.

1. Management: Because users know what Terratest is actually doing, it is pretty easy to manage test code.

### Using Terratest in Simplest Way

#### Set up Environment

To start, we need to install Terraform and Golang including Dep. Make sure that they are in `PATH`. The project should be checked out into `GOPATH`. Of course, we also need to configure terraform for Azure.

- [Terraform **(~> 0.11.7)**](https://www.terraform.io/downloads.html)
- [Golang **(~> 1.10.3)**](https://golang.org/dl/)
- [Dep **(~> 0.5.0)**](https://github.com/golang/dep)

#### Provide Terraform Files

First, we have to provide infrastructure code in root. These terraform files should represent which kind of resources we would like to test, including `main.tf`, `variables.tf`, `outputs.tf`, etc.

- [Terraform Index Page](https://www.terraform.io/)
- [Terraform Modules for Azure](https://registry.terraform.io/browse?provider=azurerm)
- [Terraform Tutorial on Azure](https://docs.microsoft.com/en-us/azure/terraform/)
- [Configure Terraform for Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/terraform-install-configure)

#### Design Test Case

In order to test infrastructure code, we have to provide test cases in `./test/fixture` folder. These test cases are also terraform files, which indicate usage of resources or modules we would like to test.

#### Write Test Code

One of the most important things we have to do is taking advantage of terratest to write test code. In `./test` folder, we should create a file named `xxx_test.go`. Inside this file, we need to write test code using Golang. With help of terratest, we can create or destroy a resource, or do any kinds of validation we like. Terratest provides functions that SSH to a virtual machine or make HTTP requests to a server. To discover more details and features, please refer to Terratest official website.

- [Terratest Source Code & Document](https://github.com/gruntwork-io/terratest/)

#### Use Dep as Package Management

Because it is quite likely for test code to introduce lots of go packages, it is better for us to use package management tools. In root, run `dep init` to generate two files `Gopkg.toml` and `Gopkg.lock`. Then run `dep ensure` to guarantee that related go packages are installed.

#### Run the Test

After we make sure that the environment has been set up correctly and we have each file defined, we can run `go test -v ./test` to start our test workflow.

### Using Terratest by Rakefile

#### Set up Environment

If we would like to utilize `Rakefile` and regard testing as a ruby task, besides Terraform and Golang, we also need to install Ruby and Bundler.

- [Ruby **(~> 2.3)**](https://www.ruby-lang.org/en/downloads/)
- [Bundler **(~> 1.15)**](https://bundler.io/)

Therefore, we provide a simple script to quickly set up module development environment:

```sh
$ curl -sSL https://raw.githubusercontent.com/Azure/terramodtest/master/tool/env_setup.sh | sudo bash
```

After we run this script, we have to define `PATH` and `GOPATH` by our own. Of course, we are still required to provide terraform files, design test case and write test code.

#### Install Gem Packages

To install gem packages (mostly terramodtest in this scenario), we may use `Gemfile` in root to define which packages we actually need.

#### Define Tasks

The essential step is to define tasks in `Rakefile`. We need to illustrate what kinds of jobs we would like to do when testing. For instance, we may define `rake build`, whose feature is to check format of terraform files, or define `rake e2e`, in order to run tests using terratest.

#### Run Rakefile

After we ensure that the environment has been set up correctly and each file is well provided, first we run `bundle install` to fetch required gem packages, and then run `rake build` and `rake e2e` (or other types of rake tasks) to see whether the infrastructure code passes the test.

### Using Docker to Run Tests

#### Set up Environment

Some people do not want to install too many SDKs or packages, which may interfere with their development environment. Thus, we may want to define a `Dockerfile` to build a new image based from `microsoft/terraform-test`.

- [Docker](https://www.docker.com/community-edition#/download)

We do not need to install anything else if we use docker, which means we can ignore setting up local environment. Nevertheless, in order to make the workflow works, writing `Gemfile` and `Rakefile` are still required.

#### Design Dockerfile

`Dockerfile` stands for configuration of a docker. We need to provide environment variables and all necessary commands, such as installing dep and gem packages.

#### Run Docker

First build the docker image.

```sh
$ docker build --build-arg BUILD_ARM_SUBSCRIPTION_ID=$ARM_SUBSCRIPTION_ID --build-arg BUILD_ARM_CLIENT_ID=$ARM_CLIENT_ID --build-arg BUILD_ARM_CLIENT_SECRET=$ARM_CLIENT_SECRET --build-arg BUILD_ARM_TENANT_ID=$ARM_TENANT_ID -t template .
```

Then run tests.

```sh
$ docker run --rm template /bin/bash -c "bundle install && rake full"
```

### Using Travis CI for Continuous Integration

#### Introduce Travis CI

Once we update a module, we would like to test the infrastructure code before release. Because testing manually is pretty cumbersome and waste of time, we may introduce travis CI to help us automatically test the module.

- [Travis CI](https://travis-ci.org/)

#### Write Script

Now we need to write a file named `.travis.yml` in root. It demonstrates what the system should do in continuous integration. One popular usage is utilizing docker to run the test, just like the previous part.

#### Run Build

After we guarantee that Travis CI has been installed for specific GitHub repository and environment variables are set correctly, the test will be run automatically once we have changed something in the repo.

### Template

The rest files of this guideline composes a template which stands for a simple usage of terratest. Although it seems quite basic, it contains almost everything required including terraform files, test case, test code, `Gemfile`, `Rakefile`, `Dockerfile`, `.travis.yml`, etc. We can conveniently run this template after we set up the environment correctly or use docker instead.

### Example

Azure Terraform Compute Module is an essential module that uses terratest. All files are defined properly which can be easily referred if we would like to introduce terratest to other modules.

- [Azure Terraform Compute Module](https://github.com/Azure/terraform-azurerm-compute)
