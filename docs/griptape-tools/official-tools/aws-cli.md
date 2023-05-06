# AwsCli

This tool enables LLMs to run AWS CLI commands. Before using this tool, make sure to [install and configure](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) AWS CLI v2.

The **AwsCli** tool uses the following parameters: 

| Parameter      | Description                          | Required |
| ----------- | ------------------------------------ |----------|
| `aws_access_key_id`  | The AWS Access key for the CLI  | YES |
| `aws_secret_access_key`       | The Secret associated with the AWS Access key | YES |
| `aws_default_region`    | The AWS region in which to the tool should run commands. Defaults to `us-east-1` | NO |
| `aws_cli_policy` | You can pass a JSON IAM Policy the tool should use | YES |

### Quickstart

There are a few important things to note. 

First, this policy below is a **terrible idea**. Use something more restrictive! We define it this way purely for illustrative purposes. The policy is passed to the LLM and used as a reference. This does not mean that it is enforced or matches the policy you have attached to this user. In practice, this policy should match the one you're using in AWS. 

Second, GPT-4 does a much better job deciphering how to use the AWS CLI. Finally, pay attention to the chain of thought output. It sometimes misuses the API, then execute some help statements to correct itself.

Here are some things you can try:

  *"what S3 buckets do i have?"
  * "what was my aws bill like the last few months?" - remember to add the [calculator](calculator.md) tool as well if you want it to roll up any billing summaries

```python
import json
from decouple import config
from griptape.tools import AwsCli
from griptape.memory import Memory
from griptape.tasks import ToolkitTask
from griptape.structures import Pipeline
from griptape.core import ToolLoader
from griptape.drivers import OpenAiPromptDriver

aws_cli = AwsCli(
    aws_access_key_id=config("AWS_ACCESS_KEY_ID"),
    aws_secret_access_key=config("AWS_SECRET_ACCESS_KEY"),
    aws_cli_policy="""{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}"""
)

pipeline = Pipeline(
    memory=Memory(),
    prompt_driver=OpenAiPromptDriver(
        model="gpt-4"
    ),
    tool_loader=ToolLoader(
        tools=[aws_cli]
    )
)

pipeline.add_tasks(
    ToolkitTask(
        tool_names=[aws_cli.name]
    )
)

result = pipeline.run("what S3 buckets do i have?")
print(result.output.value)

```

You should see a final result similar to the following: 

```
Input: what S3 buckets do i have?   
[chain of thought output... will vary depending on the model driver you're using]
You have the following S3 buckets: bucket-name, other-bucket-name, final-bucket-name.
```
