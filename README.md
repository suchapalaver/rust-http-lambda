# Rust HTTP Lambda

`rust-http-lambda` is for testing response times of lambda running on Rust.

## Building and Deploying

The easiest way to start writing Lambda functions with Rust is by using [Cargo Lambda](https://www.cargo-lambda.info/),
a project related to the [`aws-rust-lambda-runtime`](https://github.com/awslabs/aws-lambda-rust-runtime) project. Cargo
Lambda is a Cargo plugin, or subcommand, that provides several commands to help you in your journey with Rust on AWS
Lambda.

>The preferred way to install Cargo Lambda is by using a package manager.

### Use Homebrew on MacOS:

```terminal
brew tap cargo-lambda/cargo-lambda
brew install cargo-lambda
```

### Use Scoop on Windows:

```terminal
scoop bucket add cargo-lambda https://github.com/cargo-lambda/scoop-cargo-lambda
scoop install cargo-lambda/cargo-lambda
```

### Or PiP on any system with Python 3 installed:

```terminal
pip3 install cargo-lambda
```

### Zig!

Cargo Lambda is written in Zig, which I was able to install and get things working using `asdf`.

## Building for Amazon Linux 2

This function was created using the command `cargo lambda new rust-http-lambda`.

It is recommended to use Amazon Linux 2 runtimes (such as provided.al2) as much as possible for building Lambda
functions in Rust. To build your Lambda functions for Amazon Linux 2 runtimes, from inside the `rust-http-lambda`
module, run:

```terminal
cargo lambda build --release --arm64
```

There are other ways of building your function: manually with the AWS CLI, with
[AWS SAM](https://github.com/aws/aws-sam-cli), and with the [Serverless framework](https://serverless.com/framework/).

## Deploying to AWS Lambda

For a [custom runtime](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-custom.html), AWS Lambda looks for an
executable called `bootstrap` in the deployment package zip. Rename the generated executable to `bootstrap` and add it to a
zip archive.

You can find the `bootstrap` binary for your function under the `target/lambda` directory.

### Deploying with Cargo Lambda

Use the `deploy` subcommand to upload your function to AWS:

```terminal
cargo lambda deploy \
  --enable-function-url \
  --iam-role arn:aws:iam::XXXXXXXXXXXXX:role/your_lambda_execution_role
  --tags team=hewmen \
  rust-http-lambda
```

This command will create a Lambda function with the same name of your rust package.

**Make sure to replace the execution role with an existing role in your account!**

See other deployment options in [the Cargo Lambda documentation](https://www.cargo-lambda.info/commands/deploy.html).

#### [Lambda Execution Role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)

If you run the `cargo lambda deploy` command without any flags, Cargo Lambda will try to create an execution role with
Lambda's default service role policy `AWSLambdaBasicExecutionRole`.

The `AWSLambdaBasicExecutionRole` contains the following permissions:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```

### Invoking

You can test the function with the [invoke subcommand](https://www.cargo-lambda.info/commands/invoke.html):

```terminal
cargo lambda invoke --remote \
  --data-ascii '{"command": "hi"}' \
  --output-format json \
  my-first-lambda-function
```

### Deploying with the AWS CLI

You can also use the AWS CLI to deploy your Rust functions. First, you will need to create a ZIP archive of your
function. Cargo Lambda can do that for you automatically when it builds your binary if you add the `output-format`
flag:

```terminal
cargo lambda build --release --arm64 --output-format zip
```

You can find the resulting zip file in `target/lambda/YOUR_PACKAGE/bootstrap.zip`. Use that file path to deploy your
function with the [AWS CLI](https://aws.amazon.com/cli/):

```terminal
$ aws lambda create-function --function-name rustTest \
  --handler bootstrap \
  --zip-file fileb://./target/lambda/basic/bootstrap.zip \
  --runtime provided.al2023 \ # Change this to provided.al2 if you would like to use Amazon Linux 2 (or to provided.al for Amazon Linux 1).
  --role arn:aws:iam::XXXXXXXXXXXXX:role/your_lambda_execution_role \
  --environment Variables={RUST_BACKTRACE=1} \
  --tracing-config Mode=Active
```

**Make sure to replace the execution role with an existing role in your account!**

You can now test the function using the AWS CLI or the AWS Lambda console:

```terminal
$ aws lambda invoke
  --cli-binary-format raw-in-base64-out \
  --function-name rustTest \
  --payload '{"command": "Say Hi!"}' \
  output.json
$ cat output.json  # Prints: {"msg": "Command Say Hi! executed."}
```

**Note** `--cli-binary-format raw-in-base64-out` is a required argument when using the AWS CLI version 2.
[More Information](https://docs.aws.amazon.com/cli/latest/userguide/cliv2-migration.html#cliv2-migration-binaryparam)
